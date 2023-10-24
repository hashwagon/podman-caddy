# Podman Caddy
This is a `podman-compose` ([About Podman](https://docs.podman.io/en/latest/)) turn-key solution that provides [Caddy](https://caddyserver.com/) reverse proxy and automatic [Let's Encrypt HTTPS](https://letsencrypt.org/) configuration functionality, all while playing nice in a [rootless podman environment](https://github.com/containers/podman/blob/main/docs/tutorials/rootless_tutorial.md).

# Configuration

## Caddyfile
Follow the [Caddyfile Documentation](https://caddyserver.com/docs/caddyfile) to create a Caddyfile. The example Caddyfile below is simplistic, allowing for global configuration while also importing matching `*.caddy` configurations which are live hot-swappable.
    
    # Caddy
    #
    # web server | reverse proxy | automatic https
    #
    # https://caddyserver.com/docs/caddyfile
    #
    # all other web containers require their own SERVICENAME.caddy file
    # that is placed in the /podman/caddyfile.d/ directory.
    #
    #
    # global options: https://caddyserver.com/docs/caddyfile/concepts#structure
    {
            email youremail@example.com
    }
    
    # include SERVICENAME.caddy files as described above.
    import "*.caddy"
    
The `compose.yml` in this repository expects additional `*.caddy` files to be in the `/podman/caddyfile.d/` directory which mounts to `/etc/caddy` within the caddy container. See the Additional Containers section below an example file.

## Firewall
This configuration requires port 80 to be opened in the firewall.

## Create Caddy Network
Important: Don't forget to create a caddy network!

    $ podman network create caddy
    
# Additional Containers
When adding additional containers via separate compose.yml files you will need to do the following.

## Container service.caddy
Unless specified in the primary Caddyfile, the compose.yml in this repository expects a container-specific `yourcontainername.caddy` files to be in the `/podman/caddyfile.d/` directory. This directory volume mounts to `/etc/caddy` within the caddy container. Here's an example.

    reverse_proxy * YOURCONTAINERNAME:80 {
            header_up X-Forwarded-Port 443
            header_up X-Forwarded-Proto https
    
## Define Network in Compose File
In the service's compose.yml add the following at the bottom.
    
    Virtual host compose.yml contents:
    networks:
      default:
        external:
          name: caddy
    
