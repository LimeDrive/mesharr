# mesharr
 Mesharr is a synchronization tool designed to manage multiple instances of Radarr, Sonarr, Readarr, and Lidarr for **collaborative cloud storage utilization with a shared library**. This innovative application ensures seamless coordination across local services, propagating any additions or changes made to one instance to all others in the network. When a file is downloaded by one instance, it's automatically marked as monitored locally and unmonitored on the other instances, while still being present in their respective libraries.

Mesharr goes beyond synchronization by also handling metadata integration into Plex by using autoscan and optimizing cloud uploads via rclone. This synergy guarantees efficient file availability for each user, maximizing accessibility and minimizing wait times. With Mesharr, cloud-based media libraries become effortlessly harmonized, streamlining the user experience across multiple platforms.

---

### Prerequisites:

Before getting started, ensure the following prerequisites are in place:

- Existing services: Radarr, Sonarr, Readarr, Lidarr instances should already be set up.
- Multi-user merged library on the cloud: A unified library accessible by multiple users, stored in the cloud.
- Consistent configuration: All instances (arr services) should have consistent quality profiles and file naming conventions.
- User setup: Every user joining the mesh needs to configure their apps from scratch and connect an instance of Mesharr to participate.
- Cloud integration: All mesh participants must have a functional mergerfs mount of the cloud storage, and there should be agreement on the file paths.

To establish connections between instances, Mesharr employs tokens generated via command-line instructions.

Configuration is carried out through YAML files, with connections to the arr services and Plex established through local APIs (within Docker environments).

Mesharr is designed to run within a Docker container on each user's installation, aligned with their respective shared library. To streamline error management, such as instances of users being offline, Mesharr utilizes a database to store relevant information.

---

### Simple Operational Example:

Consider two identical arr instances:
- Radarr-1, App-1, Plex-1, autoscan-1 from User-1
- Radarr-2, App-2, Plex-2, autoscan-2 from User-2

Example scenario for importing/updating a new movie on **Radarr-1**:

1. **Radarr-1** sends a notification to **App-1** upon import/update/remove (via webhook or custom script).
2. **App-1** triggers a manual scan on **autoscan-1** to add/update the file on **Plex-1**.
3. **App-1** uploads the file to the cloud (copy or move) (or invokes cloudplow, although it might be complex).
4. **App-1** notifies **App-2** (with file details).
5. **App-2**, having received the notification, initiates a manual scan on **autoscan-2** to add/update the file on **Plex-2**.
6. **App-2** adds the film to **Radarr-2** as *unmonitored* through the API.

---

### Development:

- FastAPI
- Celery for task queue management
- Redis as a broker and backend (consider RabbitMQ or SQLite3 for lighter options)
- Configuration via environment variables, securing FastAPI with tokens, potential IP whitelist with Traefik

### Work-in-Progress Development (WIP-DEV):

- Single container with FastAPI Python app
- Celery with SQLite3 as backend and broker
- Security via an API key in a linked database
- The key invokes a token used to secure routes

### Work-in-Progress Steps (wipst):

- Adding a tag to files added by the app to identify the instance (and user) and monitor files
- Implementing notification senders for different situations

### Docker Deployment:

**1st Solid Option:** Deployment using a simplified Compose setup with 3 containers:
- Web: Exposing the API (possible Flower and/or UI to balance the front)
- Worker: Handling tasks, managing rclone uploads, scheduling file tasks
- Redis: Broker and backend for Celery

**2nd Option:** A more complex approach with a single container:
- Web and worker within the same app
- Broker in a database for tasks