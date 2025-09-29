using UnityEngine;
using UnityEngine.AI;
using System.Collections.Generic;

namespace Avoider
{
    [RequireComponent(typeof(NavMeshAgent))]
    public class AvoiderClass : MonoBehaviour
    {
        [Header("Avoider Settings")]
        public GameObject avoidee;
        [SerializeField] private float detectionRange = 10f;
        [SerializeField] private float escapeSpeed = 5f;
        [SerializeField] private bool drawGizmos = true;

        private NavMeshAgent m_agent;
        private bool canSee = false;

        void Start()
        {
            m_agent = GetComponent<NavMeshAgent>();

            if (avoidee == null)
                Debug.LogWarning("Must have object to avoid.");

            Debug.Log("Avoider DLL loaded.");
        }

        void Update()
        {
            if (avoidee == null) return;

            float distance = Vector3.Distance(transform.position, avoidee.transform.position);
            if (distance > detectionRange) return; // only react if within range

            CheckVisibility();

            if (canSee)
                RunAndHide();
        }

        private void CheckVisibility()
        {
            Vector3 direction = (avoidee.transform.position - transform.position).normalized;
            float dist = Vector3.Distance(transform.position, avoidee.transform.position);

            RaycastHit[] hits = Physics.RaycastAll(transform.position, direction, dist);
            canSee = false;

            foreach (var hit in hits)
            {
                if (hit.transform == avoidee.transform)
                {
                    canSee = true;
                    break;
                }
            }
        }

        private void RunAndHide()
        {
            List<Vector3> candidates = GenerateHidingSpots();

            if (candidates.Count == 0) return;

            Vector3 bestSpot = candidates[0];
            float bestDist = Vector3.Distance(transform.position, bestSpot);

            foreach (var spot in candidates)
            {
                float d = Vector3.Distance(transform.position, spot);
                if (d < bestDist)
                {
                    bestSpot = spot;
                    bestDist = d;
                }
            }

            m_agent.speed = escapeSpeed;
            m_agent.SetDestination(bestSpot);
        }

        private List<Vector3> GenerateHidingSpots()
        {
            List<Vector3> validSpots = new List<Vector3>();
            var sampler = new PoissonDiscSampler(10, 10, 2f); // area size 10x10, radius 2

            foreach (var point in sampler.Samples())
            {
                Vector3 worldPoint = transform.position + new Vector3(point.x, 0, point.y);

                Vector3 dir = (worldPoint - avoidee.transform.position).normalized;
                float dist = Vector3.Distance(worldPoint, avoidee.transform.position);

                if (Physics.Raycast(avoidee.transform.position, dir, out RaycastHit hit, dist))
                {
                    if (hit.transform != this.transform)
                        validSpots.Add(worldPoint); // hidden spot
                }
            }

            return validSpots;
        }


        private void OnDrawGizmos()
        {
            if (!drawGizmos || avoidee == null) return;

            // Detection range
            Gizmos.color = Color.red;
            Gizmos.DrawWireSphere(transform.position, detectionRange);

            int numPoints = 20; // number of lines
            float radius = 5f;  // distance of lines from avoider

            for (int i = 0; i < numPoints; i++)
            {
                float angle = i * Mathf.PI * 2f / numPoints;
                Vector3 point = transform.position + new Vector3(Mathf.Cos(angle), 0, Mathf.Sin(angle)) * radius;

                Vector3 dir = (point - avoidee.transform.position).normalized;
                float dist = Vector3.Distance(point, avoidee.transform.position);

                bool hidden = false;
                if (Physics.Raycast(avoidee.transform.position, dir, out RaycastHit hit, dist))
                {
                    if (hit.transform != this.transform)
                        hidden = true;
                }
                else
                {
                    hidden = true;
                }

                Gizmos.color = hidden ? Color.green : Color.red;
                Gizmos.DrawLine(transform.position, point);
                Gizmos.DrawSphere(point, 0.1f);
            }
        }

    }


    public class PoissonDiscSampler
    {
        private readonly float radius;
        private readonly float radius2;
        private readonly float cellSize;
        private readonly int gridWidth, gridHeight;
        private readonly Vector2[,] grid;
        private readonly List<Vector2> activeSamples = new List<Vector2>();
        private readonly List<Vector2> samples = new List<Vector2>();
        private readonly System.Random random = new System.Random();

        public PoissonDiscSampler(float width, float height, float radius)
        {
            this.radius = radius;
            this.radius2 = radius * radius;
            this.cellSize = radius / Mathf.Sqrt(2);

            gridWidth = Mathf.CeilToInt(width / cellSize);
            gridHeight = Mathf.CeilToInt(height / cellSize);
            grid = new Vector2[gridWidth, gridHeight];

            AddSample(new Vector2(width / 2, height / 2));
        }

        public IEnumerable<Vector2> Samples()
        {
            return samples;
        }

        private void AddSample(Vector2 sample)
        {
            samples.Add(sample);
            activeSamples.Add(sample);
            grid[(int)(sample.x / cellSize), (int)(sample.y / cellSize)] = sample;
        }
    }
}


