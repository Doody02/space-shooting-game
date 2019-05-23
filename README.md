# space-shooting-game
this is the code for my space shooting game runs by unity 
// player controller //
public class PlayerController : MonoBehaviour {
public float speed = 15.0f;
public float padding = 1f;
public GameObject projectile;
public float projectileSpeed;
public float firingRate = 0.2f;
public float health = 250f;

public AudioClip fireSound;

float xmin;
float xmax;

void Start(){

float distance = transform.position.z - Camera.main.transform.position.z;
Vector3 leftmost = Camera.main.ViewportToWorldPoint(new Vector3(0,0,distance));
		Vector3 rightmost = Camera.main.ViewportToWorldPoint(new Vector3(1,0,distance));
		xmin = leftmost.x + padding;
		xmax = rightmost.x - padding;

}
void Fire(){
Vector3 offset = new Vector3(0,1,0);
		GameObject beam = Instantiate(projectile , transform.position+offset  , Quaternion.identity) as GameObject ;
		beam.rigidbody2D.velocity = new Vector3(0,projectileSpeed,0);
		AudioSource.PlayClipAtPoint(fireSound, transform.position);
		}
	void Update () {
	if(Input.GetKeyDown(KeyCode.Space)){
	InvokeRepeating ("Fire", 0.000001f, firingRate);
	}
	
	if(Input.GetKeyUp(KeyCode.Space)){
	CancelInvoke("Fire");
	}
	if(Input .GetKey (KeyCode.LeftArrow)){
	transform.position += Vector3.left * speed * Time.deltaTime;
		}else if(Input .GetKey (KeyCode.RightArrow)){
			transform.position +=  Vector3.right * speed * Time.deltaTime;
		}	
		
		//restric the player to the gamespace
		float newX = Mathf.Clamp (transform.position.x, xmin, xmax);
		transform.position = new Vector3(newX, transform.position.y, transform.position.z);
	}
	void OnTriggerEnter2D(Collider2D collider){
		
		projectile missile = collider.gameObject.GetComponent<projectile>();
		if(missile){
			Debug.Log ("Player Collided with missile");
			health -= missile.GetDamage ();
			missile.Hit();
			 if (health <= 0){
			 Die();
			 
				
				
			}
		}
	}
	void Die(){
		LevelManager man = GameObject.Find("LevelManager").GetComponent<LevelManager>();
		man.LoadLevel("win Screen");
		Destroy(gameObject);
	}
}

// enemy behavior //
public class EnemyBehavior : MonoBehaviour {
public float projectileSpeed = 10;
public GameObject projectile;
public float health = 150;
public float shotsPerSeconds = 0.5f;
public int scoreValue = 150;

public AudioClip fireSound;
public AudioClip deathSound;

private ScoreKeeper scoreKeeper;

void Start(){
scoreKeeper = GameObject.Find("Score").GetComponent<ScoreKeeper>();
}

void Update (){
float probability = Time.deltaTime * shotsPerSeconds ;
if (Random.value < probability ){
Fire ();
}

}
void Fire(){
		Vector3 startPosition = transform.position + new Vector3(0, -1 , 0);
		GameObject missile = Instantiate(projectile, startPosition  , Quaternion.identity)as GameObject ;
		missile.rigidbody2D.velocity = new Vector2(0 , -projectileSpeed);
		AudioSource.PlayClipAtPoint(fireSound, transform.position);
		}
	void OnTriggerEnter2D(Collider2D collider){
	Debug.Log ("Enemy was hit by " + collider.gameObject.name);
	projectile missile = collider.gameObject.GetComponent<projectile>();
	if(missile){
	health -= missile.GetDamage ();
	missile.Hit();
	if (health <= 0){
	AudioSource.PlayClipAtPoint(deathSound, transform.position);
	Destroy(gameObject);
	scoreKeeper.Score(scoreValue);
	
	
	}
	}
	}
}

// enemy spawner //
public class EnemySpawner : MonoBehaviour {
public GameObject enemyPrefab;
public float width = 10f;
public float height = 5f;
public float speed = 5f;
public float spawnDelay = 0.5f;
	private bool movingRight = true;
	private float xmax;
	private float xmin;
	// Use this for initialization
	void Start () {
	float distanceToCamera = transform.position.z - Camera.main.transform.position.z;
		Vector3 leftBoundary = Camera.main. ViewportToWorldPoint (new Vector3(0,0, distanceToCamera));
		Vector3 rightBoundary = Camera.main. ViewportToWorldPoint (new Vector3(1,0, distanceToCamera));
		xmax = rightBoundary.x ;
		xmin = leftBoundary.x ;
		SpawnUntilFull();
	}
	void SpawnEnemies(){
		foreach(Transform child in transform){
			GameObject enemy = Instantiate(enemyPrefab , child.transform.position, Quaternion.identity) as GameObject;
			enemy.transform.parent = child;
		}
	
	}
	void SpawnUntilFull(){
	Transform freePosition = NextFreePosition();
	if(freePosition){
		GameObject enemy = Instantiate(enemyPrefab , freePosition.position, Quaternion.identity) as GameObject;
		enemy.transform.parent = freePosition;
		}
		if(NextFreePosition()){
		Invoke ("SpawnUntilFull", spawnDelay);
		}
	}
	public void OnDrawGizmos(){
	Gizmos.DrawWireCube(transform.position, new Vector3(width, height));
	}
	
	
	// Update is called once per frame
	void Update () {
	if(movingRight){
	transform.position += Vector3.right * speed * Time.deltaTime;
	}
	else{
			transform.position += Vector3.left * speed * Time.deltaTime;
			}
			float rightEdgeOfFormation = transform.position.x + (0.5f*width);
		float lefttEdgeOfFormation = transform.position.x - (0.5f*width);
		if(lefttEdgeOfFormation < xmin  ){
		movingRight = true;
		}
		else if(rightEdgeOfFormation > xmax){
		movingRight = false;
		}
		if (AllMembersDead()){
		Debug.Log ("Empty Formation");
			SpawnUntilFull();
			}
			}
			Transform NextFreePosition(){
			foreach (Transform childPositionGameObject in transform){
				if (childPositionGameObject.childCount == 0){
					return childPositionGameObject;
					
				}
			} 
			return null ;
	}
	bool AllMembersDead(){
	foreach (Transform childPositionGameObject in transform){
	if (childPositionGameObject.childCount > 0){
	return false ;
	
	}
	} 
	return true ;
}
}
// position //
public class position : MonoBehaviour {

	// Use this for initialization
void OnDrawGizmos(){
Gizmos.DrawWireSphere(transform.position,1);
}
}

// scripts_LevelManager //
public class LevelManager : MonoBehaviour {

	public void LoadLevel(string name){
		Debug.Log ("New Level load: " + name);
		Application.LoadLevel (name);
	}

	public void QuitRequest(){
		Debug.Log ("Quit requested");
		Application.Quit ();
	}

}
// scripts_MusicPlayer //
public class MusicPlayer : MonoBehaviour {
	static MusicPlayer instance = null;
	
	void Start () {
		if (instance != null && instance != this) {
			Destroy (gameObject);
			print ("Duplicate music player self-destructing!");
		} else {
			instance = this;
			GameObject.DontDestroyOnLoad(gameObject);
		}
		
	}
}
// scripts_projectile //
public class projectile : MonoBehaviour {
public float damage = 100f;

public float GetDamage(){
return damage;
}
public void Hit(){
Destroy(gameObject);
}
	
}
// scripts_shredder //
public class Shredder : MonoBehaviour {

	void OnTriggerEnter2D(Collider2D col){
	Destroy(col.gameObject );
	}
	
	}
  // NewBhaviorScript //
  public class NewBehaviourScript : MonoBehaviour {

	// Use this for initialization
	void Start () {
	
	}
	
	// Update is called once per frame
	void Update () {
	
	}
}
// ScoreDisplay //
public class ScoreDisplay : MonoBehaviour {

	// Use this for initialization
	void Start () {
	Text myText = GetComponent<Text>();
	myText.text = ScoreKeeper.score.ToString();
	ScoreKeeper.Reset();
	
	}
	
	// Update is called once per frame
	void Update () {
	
	}
}
// ScoreKeeper //
public class ScoreKeeper : MonoBehaviour {

public static int score = 0;
private Text myText;

void Start(){
myText = GetComponent<Text>();
Reset();
}

public void Score(int points){
Debug.Log ("Scored points");
score += points;
myText.text = score.ToString();
}
public static void Reset(){
score = 0;
	
}
}
