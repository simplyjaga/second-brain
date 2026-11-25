# Central Limit Theorem and Redis key eviction

Capture: CLT and how it's used in Redis for key eviction.

- Redis approximates LRU/LFU by sampling a few keys and evicting the best candidate — why does sampling work well? Connection to CLT / approximate eviction.
- Where does this sit? Probably [[Redis Internals]] journey note + a CLT note under Mathematics.

## Questions / Things to explore

- How does Redis's random sampling approximate true LRU? What role does CLT/distribution of sampled ages play?
