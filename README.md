using UnityEngine;
using UnityEngine.AI;



namespace Avoider
{
    public class AvoiderClass:MonoBehaviour
    {
        private NavMeshAgent m_agent;//navmeshagent for the avoider
        public GameObject avoidee;// Object you want to avoid
        public bool canSee = false;

        public float radius = 1f; // range for raycast
        void Start()
        {
            m_agent = GetComponent<NavMeshAgent>();

            if (m_agent == null) // if no navemeshagent debugWarning
            {
                Debug.LogWarning(" make the object   a NavMesh Agent and bake a NavMesh.");
                return;
            }
            if (avoidee == null)// if no avoidee object debugWarning
            {
                Debug.LogWarning("Must have object to avoid");
            }
            Debug.Log("DLL works");
        }

        void Update()
        {

        }

        public void PoissonDiscSampler(float width, float height, float radius) // radius will be how are away the lines draw from the avoider?
        {

        }

        public void Visiblity(GameObject avoider, GameObject avoidee) // Check if avoider can be seen by avoidee
        {
            Vector3 startPos = avoider.transform.position;
            Vector3 targetPos = avoidee.transform.position;
            Vector3 direction = (targetPos - startPos).normalized;

            float distance = Vector3.Distance(startPos, targetPos);

            if (Physics.Raycast(startPos, direction, out RaycastHit hit, distance))
            {
                if (hit.transform == avoidee)
                {
                    Debug.Log("Avoidee can see avoider");
                    canSee = true;
                }
                else
                {
                    Debug.Log("Something in the way");
                    canSee = false;
                }

            }
        }

        public void CheckFunction()
        {
            Visiblity(this.gameObject, avoidee);
            if (canSee)
            {
                Debug.Log("Can see");
                //Check if there is a place to run using PoissonDiscSampler
                //if yes tell agent to move (Point stored from PossionDiscSampler
                //if no wait and check again
            }
            else if (!canSee)
            {
                return;// run function again
            }
        }
    }
}
