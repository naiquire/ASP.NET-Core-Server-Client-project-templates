# [ASP.NET Core] Server/Client project templates
Project templates for setting up a client-server connection
- Handles multiple connections from a single client
- Connects to a subdomain of a server, and uses NGINX to forward requests to correct ports.
Note that installation of NGINX is not included in these templates
### Installation
Drag the .zip files into `C:\Users\[USERNAME]\Documents\[VISUAL STUDIO VERSION]\Templates\ProjectTemplates`
## Client template
The client is set up to make requests to an IP address with port number 5252, which can be changed to any port as long as it's updated throughout. The IP addresses (lines 24-26) should be set to the public IP address of the server, or a FQDN. The `subdomain` is the server application which the request is forwarded to. This is unlikely to differ for each connection.

As an example, suppose a messaging application is running at path `/message-app` - the client application will make requests to `/messaging-app`. A specific class can then be called such as "account" or "messages" using `/messaging-app/account`. These aren't strictly necessary, but having all the requests go into one class could become hard to maintain. It's also possible to add classes within these classes (`/messaging-app/account/create`) if required, or to improve readability.

For each of these unique URLs, there will need to be an new `HubConnection` added to the `_connections` structure, and the calls for `initialiseConnection` and `startConnection` will need to be updated. An handle for a connection loss will also need to be added in the _Handle connection loss_ region. The handles for each connection are techically linked for all connections - this is to make handle creation more convinient. Each connection should be set up to call a specific class on the server, such that the server calling these handles will only result in one connection triggering.
## Server template
Each `subdomain` is linked to the unique port number for the current instance of the server application. NGINX should be set up to forward requests on :5252/[subdomain] to the port identified with that subdomain. Done correctly, the subdomains in the URL paths should all be the same. The classes and subclasses are available for calling a specific class in `Subdomains/class.cs`. Line 61 should not have its IP address changed, and should remain as `0.0.0.0`

For each request that the client makes to a specific `subdomain`, there will need to be an endpoint added for it in `Program.cs`. The structure of the classes is self-explanatory. The subclasses are unlikely to be required and would in most cases be replaced with subroutines. But the option is still there if it's required.

### NGINX
Installation:
```
sudo apt install nginx
sudo nano /etc/nginx/sites-available/default
```
Here is an example NGINX configuration:
```
server {
    listen {port number};

    server_name {server address};

    location /app1 {
        proxy_pass http://localhost:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /app2 {
        proxy_pass http://localhost:5001;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```
