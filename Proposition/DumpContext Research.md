
## With existing connectivityContext

- **Pros** :
	- Faster to develop
	- Less complexity
- **Cons** :
	- Heavy burden on dumb connectivityContext
	- New service is not decoupled regarding data
	- expiration

## With Event Driven

- **Pros** :
	- Clearly decoupled
	- Responsibility over its own data
	- Can easily add more services that use that
- **Cons** :
	- Complexity
		- Event Scheme
		- Understanding of architecture
	- Need more time to develop

---

![[17-09-24 16.19.excalidraw|700]]