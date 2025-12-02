MongoDB 4.4 Replica Set on Docker (Non-AVX Compatible)

This guide helps you deploy a 3-node MongoDB 4.4 Replica Set using Docker Compose. It is specifically designed for systems without AVX CPU support, where MongoDB 5.x+ cannot run.

⸻

✅ Features
	•	MongoDB 4.4 (last version without AVX requirement)
	•	3-node Replica Set
	•	Persistent volumes
	•	Explicit replica set name: myReplicaSet
	•	Works on a single Docker host
	•	Suitable for development and staging

⸻

⚙️ Prerequisites
	•	Docker installed
	•	Docker Compose v2+
	•	Minimum 2 GB RAM recommended

Verify installation:

docker --version
docker compose version


⸻

📁 Project Structure

.
├── docker-compose.yml
└── README.md


⸻

📄 docker-compose.yml

Create a file named docker-compose.yml with the following contents:

version: "3.8"

services:
  mongo1:
    image: mongo:4.4
    container_name: mongo1
    hostname: mongo1
    command:
      - mongod
      - --replSet
      - myReplicaSet
      - --bind_ip_all
    ports:
      - "27017:27017"
    volumes:
      - mongo1-data:/data/db
    networks:
      - mongoCluster

  mongo2:
    image: mongo:4.4
    container_name: mongo2
    hostname: mongo2
    command:
      - mongod
      - --replSet
      - myReplicaSet
      - --bind_ip_all
    ports:
      - "27018:27017"
    volumes:
      - mongo2-data:/data/db
    networks:
      - mongoCluster

  mongo3:
    image: mongo:4.4
    container_name: mongo3
    hostname: mongo3
    command:
      - mongod
      - --replSet
      - myReplicaSet
      - --bind_ip_all
    ports:
      - "27019:27017"
    volumes:
      - mongo3-data:/data/db
    networks:
      - mongoCluster

volumes:
  mongo1-data:
  mongo2-data:
  mongo3-data:

networks:
  mongoCluster:
    driver: bridge


⸻

▶️ Start the MongoDB Cluster

docker compose up -d

Verify all containers are running:

docker ps


⸻

🔧 Initialize the Replica Set

MongoDB 4.4 uses the legacy mongo shell.
Run this once:

docker exec -it mongo1 mongo --eval '
rs.initiate({
  _id: "myReplicaSet",
  members: [
    { _id: 0, host: "mongo1:27017" },
    { _id: 1, host: "mongo2:27017" },
    { _id: 2, host: "mongo3:27017" }
  ]
})
'


⸻

✅ Verify Replica Set Status

Check replica set name:

docker exec -it mongo1 mongo --eval "rs.conf()._id"

Check full status:

docker exec -it mongo1 mongo --eval "rs.status()"

Expected:
	•	mongo1 → PRIMARY
	•	mongo2 → SECONDARY
	•	mongo3 → SECONDARY

⸻

🔌 Connection String

Use this URI from your application or MongoDB client:

mongodb://localhost:27017,localhost:27018,localhost:27019/?replicaSet=myReplicaSet


⸻

🛑 Stop / Start the Cluster

Stop containers but keep data:

docker compose down

Start again with preserved data:

docker compose up -d

Destroy everything including volumes:

docker compose down -v


⸻

⚠️ Important Notes
	•	All replica set names must match exactly in:
	•	mongod --replSet myReplicaSet
	•	rs.initiate({ _id: "myReplicaSet" })
	•	Connection string ?replicaSet=myReplicaSet
	•	This setup is suitable for:
	•	Development
	•	Testing
	•	Staging
	•	It is NOT recommended for production because all nodes run on the same host.

⸻

🔐 Optional Enhancements

You can extend this setup with:
	•	Authentication (users & passwords)
	•	Keyfile-based replica set security
	•	Automated backups (mongodump + cron)
	•	Arbiter node
	•	Hidden secondary for analytics

⸻

📝 License

This setup is provided for educational and development purposes.
