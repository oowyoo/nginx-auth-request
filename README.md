# Nginx Auth Request Demo (Traefik Version)

This project demonstrates how to use Traefik to replace Nginx and implement authentication proxy based on HTTP headers.

## Project Structure

```
.
├── auth/                # Flask Auth Service
├── traefik/             # Traefik Configuration
│   ├── traefik.yml      # Traefik Static Config
│   └── dynamic/
│       └── dynamic.yml  # Traefik Dynamic Config (ForwardAuth)
├── docker-compose.yml   # Local Compose
└── README.md
```

## Traefik Auth Proxy Solution

- All requests are first forwarded by Traefik's ForwardAuth middleware to the `auth` service `/auth` endpoint for header validation
- If validation passes, the request proceeds to the backend; otherwise, a 401 is returned
- Fully replaces the original Nginx auth proxy solution, compatible with Kubernetes IngressRoute configuration concepts

## How to Start

1. Install and start Docker Desktop
2. In the project root directory, run:

   ```sh
   docker compose up --build
   ```

3. Wait for both `auth` and `traefik` services to start successfully

## How to Test

### PowerShell (Recommended)

```powershell
Invoke-WebRequest -Uri "http://localhost:8080/health"
Invoke-WebRequest -Uri "http://localhost:8080/health" -Headers @{"x-pretest"="valid-token"}
Invoke-WebRequest -Uri "http://localhost:8080/health" -Headers @{"x-pretest"="wrong-token"}
```

### Linux/macOS/curl

```sh
curl http://localhost:8080/health
curl -H "x-pretest: valid-token" http://localhost:8080/health
curl -H "x-pretest: wrong-token" http://localhost:8080/health
```

- No header or wrong header returns 401
- Correct header (`x-pretest: valid-token`) returns 200

## Traefik Dashboard

Visit [http://localhost:8081/dashboard/](http://localhost:8081/dashboard/) in your browser to view Traefik routes and middleware.

## Other Notes

- All original Nginx-related files have been removed; everything is now implemented with Traefik
- To restore the Nginx solution, refer to the commit history
- This project is for demonstration and interview testing only. For production, please use a secure WSGI server and a robust authentication solution.

## Nginx vs Traefik (Key Differences)

| Topic | Nginx (in original example) | Traefik (in this repo) |
| --- | --- | --- |
| Reverse proxy role | Nginx server blocks + locations | Routers + Services |
| Auth integration | `auth_request` to subrequest location | `ForwardAuth` middleware to external auth service |
| Dynamic config | Reload required for file changes | Hot reload via File provider (`watch: true`) |
| Kubernetes integration | Ingress (via ingress-nginx), annotations vary | Native CRDs (IngressRoute, Middleware) + Ingress support |
| Observability | Access/error logs, 3rd-party dashboards | Built-in dashboard (`:8081`) |
| Config style | Nginx directives | YAML (static + dynamic) |
| Canary/traffic shaping | Requires extra modules/ingress controller features | Built-in via services, weights, middlewares |
| TLS management | External tooling (e.g. cert-manager) in K8s | Built-in ACME support (not enabled here) |

## Why Choose ForwardAuth in Traefik

- Simplicity: Mirrors Nginx `auth_request` pattern—centralized gatekeeping with an external auth service.
- Separation of concerns: Traefik focuses on routing; the `auth` service encapsulates authentication logic.
- Reuse across routes: Apply the same middleware to multiple routers without duplicating logic.
- Works in Docker and Kubernetes: Same concept maps to Traefik's Middleware in File provider and IngressRoute CRDs.
- Observability: Auth decisions and routing can be inspected in Traefik dashboard and logs.

### Trade-offs and Alternatives
- ForwardAuth vs. JWT middleware: If you have signed JWTs and want stateless edge validation, a JWT middleware could avoid the extra auth round-trip. Here we intentionally matched the original `auth_request` design.
- Custom plugin: Traefik supports plugins, but ForwardAuth is standard and simpler to maintain.
- Performance: ForwardAuth adds a network hop; keep the auth service lightweight and colocated. 
