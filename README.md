# Photon + Nominatim + OpenSearch (Local Geocoder Setup)

This project sets up a complete local geocoding stack using:

- [Photon](https://github.com/komoot/photon)
- [OpenStreetMap Nominatim](https://nominatim.org/)
- [OpenSearch](https://opensearch.org/)
- Docker Compose

✅ Optimized for offline/local use  
🇸🇪 Example dataset: `sweden-latest.osm.pbf`  
⚙️ Supports structured searches with support for language variants

---

Here’s a clean and improved version of those two sections, with repetition removed, clarity improved, and formatting cleaned up. It also makes it more adaptable for users choosing another country than Sweden:

---

## 🌍 OSM Data Source

This project uses OpenStreetMap data via [Geofabrik downloads](https://download.geofabrik.de/), which provides regional extracts in `.osm.pbf` format.

To use a different region:

1. Visit [https://download.geofabrik.de/](https://download.geofabrik.de/)
2. Choose and download your desired region (e.g., `europe/finland-latest.osm.pbf`)
3. Place the `.osm.pbf` file in the project root
4. Update the corresponding `volumes` and `command` lines in the `nominatim-import` service in `docker-compose.yml`:

   ```yaml
   volumes:
     - ./finland-latest.osm.pbf:/nominatim/data/finland-latest.osm.pbf:ro

   command: >
     sh -c "psql -h db -U ${POSTGRES_USER} -tAc \"SELECT 1 FROM pg_roles WHERE rolname='www-data'\" | grep -q 1 || createuser -h db -U ${POSTGRES_USER} www-data &&
            nominatim import --osm-file /nominatim/data/finland-latest.osm.pbf"
   ```

---

## 🔧 Requirements

* Docker
* Docker Compose
* An `.osm.pbf` extract file in the project root

### Example: [Download Sweden](https://download.geofabrik.de/europe/sweden.html)

Be sure the filename and path are consistent throughout `docker-compose.yml` to avoid import errors.

---

## 🚀 Setup

### 1. Clone and configure

```bash
git clone https://github.com/YOUR_USERNAME/photon-local.git
cd photon-local
cp .env.example .env
```

Edit `.env` and set:

```dotenv
POSTGRES_USER=nominatim
POSTGRES_PASSWORD=nominatim
POSTGRES_DB=nominatim
OPENSEARCH_INITIAL_ADMIN_PASSWORD=admin
JAVA_OPTS=-Xms1g -Xmx1g
```

---

### 2. Run Nominatim Import (once)

```bash
docker compose up --build nominatim-import
```

> This step may take a while depending on dataset size (~30 minutes for Sweden).

---

### 3. Import into Photon

```bash
docker compose up --build photon-import
```

> If you encounter memory issues, increase `JAVA_OPTS` or allocate more Docker resources.

---

### 4. Start the Services

```bash
docker compose up -d db opensearch photon
```

Photon runs as a headless service (no web UI). Use `curl` or integrate into an application.

---

## 🔍 Usage

Query Photon:

```bash
curl "http://localhost:2322/api?q=stockholm&lang=sv"
```

Reverse lookup

```bash
curl "http:///localhost:2322/reverse?lat=59.3251172&lon=18.0710935"
```

**API Parameters**:

| Parameter | Description                                 |
|-----------|---------------------------------------------|
| `q`       | Query string (e.g., `stockholm`)            |
| `lang`    | Language code (`sv`, `en`, etc.) (optional) |
| `limit`   | Number of results (optional)                |

---

## 🧱 Directory Structure

```
.
├── docker-compose.yml
├── Dockerfile.photon
├── sweden-latest.osm.pbf     # (not in repo)
├── .env.example
└── README.md
```

---

## 🧹 Docker Cleanup (Optional)

To remove unused containers, images, and volumes:

```bash
docker system prune -a
docker volume prune
```

> ⚠️ Warning: This will remove **all unused resources** from Docker.

---

## 📄 License

MIT — see [LICENSE](./LICENSE)

---

## 🙏 Credits

- [Geofabrik](https://download.geofabrik.de/) for OSM extracts
- [Komoot](https://github.com/komoot/photon) for Photon
- [Mediagis](https://github.com/mediagis/nominatim-docker) for the Nominatim Docker image
- [OpenSearch](https://opensearch.org/) for powering full-text search

