# Automate HTTP Service Deployment

A lightweight .NET 6 HTTP service that serves random Amazon S3 facts, plus a deployment script to run it on Ubuntu with systemd and AWS CodeCommit.

## What It Does

The service listens on **port 8002** and responds to HTTP requests with:

- The current time (time of day)
- A randomly chosen fact about Amazon S3 (storage, durability, security, analytics, replication, etc.)

Each request returns a single line in the form: `HH:MM:SS - <S3 fact>`.

## Project Structure

```
.
├── main.cs          # HTTP listener and S3 facts logic
├── srv02.csproj     # .NET 6 project file
├── build.sh         # Ubuntu deployment script (systemd + CodeCommit)
└── README.md
```

## Prerequisites

- **.NET 6 SDK** (for building and running locally)
- For deployment via `build.sh`: Ubuntu, `apt`, and AWS CodeCommit access

## Running Locally

1. Install the [.NET 6 SDK](https://dotnet.microsoft.com/download/dotnet/6.0) if needed.

2. Build and run:

   ```bash
   dotnet run
   ```

   Or build then run the published app:

   ```bash
   dotnet publish -c Release --self-contained=false --runtime linux-x64
   dotnet bin/Release/netcoreapp6/linux-x64/srv02.dll
   ```

3. Send a request:

   ```bash
   curl http://localhost:8002/
   ```

   Example response: `14:32:05.123 - Scale storage resources to meet fluctuating needs with 99.999999999% (11 9s) of data durability.`

## Deployment (Ubuntu)

`build.sh` automates setup on an Ubuntu host. It:

1. Updates packages and installs **.NET 6** (runtime + SDK), **git**, **unzip**
2. Installs and configures the **AWS CLI**
3. Configures **git** for AWS CodeCommit (credential helper)
4. Clones the repo from CodeCommit (`srv-02`) into `/home/ubuntu/srv-02`
5. Publishes the app with `dotnet publish`
6. Creates a **systemd** unit `srv-02.service` and starts the service

**Note:** The script assumes:

- Run with appropriate privileges (e.g. `sudo` or as root)
- The CodeCommit repo URL and region (`eu-north-1`) match your setup
- The host is an Ubuntu system (e.g. EC2) with network access to CodeCommit

To run the deployment script:

```bash
chmod +x build.sh
sudo ./build.sh
```

After deployment, the service runs as `srv-02` and listens on port 8002. Useful commands:

```bash
sudo systemctl status srv-02   # Check status
sudo systemctl restart srv-02  # Restart
sudo systemctl stop srv-02     # Stop
```

## API

| Method | Path   | Description                    |
|--------|--------|--------------------------------|
| GET    | `/`    | Returns current time + random S3 fact |

Response: plain text, UTF-8, single line.

## License

Use and modify as needed for your environment.
