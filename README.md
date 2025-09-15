# ⚡ Bill Splitter

A tiny Flask web app that calculates two people’s monthly totals given an electric bill input and a set of required environment variables for all other costs.

---

## Features

* Simple UI: enter the electric bill, get per-person totals and the full household total.
* Production container image using Gunicorn.

---

## RUN Locally Via docker

```bash
docker run --rm -p 5000:5000 --env-file .env --name bill-splitter ghcr.io/hmalin/bill-splitter:0.1.0
```

3. Open: [http://localhost:5000](http://localhost:5000/)
    
4. Stop: `Ctrl+C` (foreground) or `docker stop bill-splitter`.
    

### Test via curl

```bash
curl -X POST -F electric=123.45 http://localhost:5000/
```

---

## Docker Compose

Create `docker-compose.yml` next to your `.env`:

```yaml
services:
  bill-splitter:
    image: ghcr.io/hmalin/bill-splitter:0.1.0  # or bill-splitter:latest if built locally
    container_name: bill-splitter
    ports:
      - "5000:5000"          # host:container
    env_file: .env            # REQUIRED: provides RENT_AMT, etc.
    restart: unless-stopped   # optional
```

### Run it:

```bash
docker compose up -d
```

### Logs & lifecycle:

```bash
docker compose logs -f bill-splitter
docker compose ps
docker compose down
```

### Upgrade to a new image tag:

```bash
docker compose pull && docker compose up -d
```



## App Structure

```
.
├── app.py                # Flask app (strict env handling)
├── templates/
│   └── index.html        # UI template
├── static/
│   └── styles.css        # Stylesheet
├── requirements.txt      # Flask + gunicorn
├── Dockerfile            # Python 3.12 slim, Gunicorn
├── .env                  # Your real config (NOT committed)
└── .env.example          # Example config (committed)
```

> Your exact layout may vary; ensure `templates/` and `static/` are present.

---

## Required Environment Variables

These **must** be set for the app to start:

| Variable         | Type   | Description                   |
| ---------------- | ------ | ----------------------------- |
| `RENT_AMT`       | float  | Total monthly rent            |
| `WASTE_AMT`      | float  | Total waste/trash bill        |
| `INTERNET_AMT`   | float  | Total internet bill           |
| `MOBILE_AMT`     | float  | Total mobile/cellular bill    |
| `PERSON_A_PHONE` | float  | Person A device/add-on charge |
| `PERSON_B_PHONE` | float  | Person B device/add-on charge |
| `PERSON_A`       | string | Person A display name         |
| `PERSON_B`       | string | Person B display name         |

---

## Development Notes

* The container uses **Gunicorn** (e.g., `-b 0.0.0.0:5000`) for production. Do not use Flask’s dev server in production.
* The app template expects keys: `person_a`, `person_b`, `person_a_total`, `person_b_total`, `bill_total`.
* Input validation: the `electric` field must be numeric and non-negative.

---

## Troubleshooting

* **App exits on start**: A required env var is missing or not numeric—check your `.env` or `ConfigMap` values.
* **Container runs but page times out**: Ensure you published the correct port (`-p 5000:5000`) and no other process is bound to 5000.
* **GHCR login error**: Use a PAT with `write:packages`; see the temporary Docker config workaround above.
* **Kubernetes not picking envs**: Confirm `envFrom.configMapRef.name` matches the created `ConfigMap`, and namespace is correct.

---

## Security & Ops Tips

* Keep `.env` out of version control.
* Treat GHCR PATs like secrets; use a local OS keychain/`pass` or GitHub Actions OIDC with `write:packages` where possible.
* For GitOps, store manifests (and non-sensitive values) in your repo; inject sensitive values (if any in the future) via `Secret` and sealed-secrets or an external secrets operator.

---

## License

MIT license
