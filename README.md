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
