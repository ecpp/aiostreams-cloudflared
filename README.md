# Self-Hosted AIOStreams

A Docker-based setup for running your own [AIOStreams](https://github.com/Viren070/AIOStreams) instance with Cloudflare Tunnel.

## Requirements

- Docker and Docker Compose
- A domain name (managed through Cloudflare)
- Cloudflare account (free tier works)

## Quick Start

```bash
# Clone and enter the directory
cd selfaiostreams

# Copy the example environment file
cp .env.example .env

# Generate a secret key (64 hex characters)
openssl rand -hex 32

# Edit .env.prod with your values
nano .env.prod

# Start everything
docker-compose -f docker-compose.prod.yml --env-file .env.prod up -d
```

## Configuration

Edit `.env.prod` with your values:

| Variable | Description |
|----------|-------------|
| `DB_PASSWORD` | PostgreSQL password (use something strong) |
| `SECRET_KEY` | 64-char hex key from `openssl rand -hex 32` |
| `BASE_URL` | Your full URL, e.g. `https://aiostreams.yourdomain.com` |
| `TUNNEL_TOKEN` | Token from Cloudflare Tunnel (see below) |

Optional settings like debrid API keys, TMDB credentials, and proxy configs are documented in `.env.example`.

## Setting Up Cloudflare Tunnel

Since this setup doesn't expose any ports directly, you'll need a Cloudflare Tunnel to make your instance accessible. The tunnel creates a secure connection between Cloudflare's network and your server - no port forwarding or firewall rules needed.

### Create the Tunnel

1. Open [Cloudflare Zero Trust Dashboard](https://one.dash.cloudflare.com/)
2. Go to **Networks > Tunnels**
3. Click **Create a tunnel**
4. Name it something like `aiostreams`
5. Skip the connector installation (we're using Docker)
6. Copy the tunnel token - you'll need this for `.env.prod`

### Configure the Public Hostname

Still in the tunnel settings:

1. Go to the **Public Hostname** tab
2. Add a public hostname:
   - **Subdomain**: `aiostreams` (or whatever you prefer)
   - **Domain**: Select your domain
   - **Service Type**: `HTTP`
   - **URL**: `aiostreams:3000`

That's it. Once your containers are running, the tunnel will connect automatically and your instance will be live at your configured domain.

## Access Control (Optional)

If you want to restrict who can access your instance, Cloudflare Access lets you add authentication. This is separate from the tunnel itself.

1. In Zero Trust, go to **Access > Applications**
2. Add a self-hosted application
3. Set the domain to match your AIOStreams URL
4. Create a policy - for example, allow only specific email addresses

Users will need to authenticate through Cloudflare before reaching your instance. This is optional but useful if you don't want the config page publicly accessible.

## Managing the Stack

```bash
# View logs
docker-compose -f docker-compose.prod.yml logs -f

# View logs for specific service
docker-compose -f docker-compose.prod.yml logs -f aiostreams

# Stop everything
docker-compose -f docker-compose.prod.yml down

# Update to latest AIOStreams image
docker-compose -f docker-compose.prod.yml pull
docker-compose -f docker-compose.prod.yml up -d

# Full reset (removes database)
docker-compose -f docker-compose.prod.yml down -v
```

## File Structure

```
selfaiostreams/
├── docker-compose.prod.yml   # Main compose file
├── .env.example              # Template for environment variables
├── .env.prod                 # Your actual config (create this)
└── data/                     # AIOStreams data (created on first run)
```

## Troubleshooting

**Tunnel not connecting**
- Check that `TUNNEL_TOKEN` is correct in `.env.prod`
- Verify the tunnel shows as "Healthy" in Cloudflare dashboard

**502 Bad Gateway**
- AIOStreams container might still be starting, wait a minute
- Check logs: `docker-compose -f docker-compose.prod.yml logs aiostreams`

**Database connection errors**
- Make sure `DB_PASSWORD` matches in your env file
- Check postgres logs: `docker-compose -f docker-compose.prod.yml logs postgres`

## Links

- [AIOStreams GitHub](https://github.com/Viren070/AIOStreams)
- [AIOStreams Docs](https://guides.viren070.me/stremio/addons/aiostreams)
- [Cloudflare Tunnel Docs](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/)
