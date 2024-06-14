# tunneling-wireguard-configs
!Warning! While this setup works, the tunneled server won't properly announce itself to Steam's master server (it wont show up in the server browser) Currently looking for a fix.

Autismobox server traffic is tunneled.

This repo contains instructions and config files to tunnel team fortress 2 game traffic.

Client (tf2 player) -> Tunneling server (VPS) -> Pterodactyl server -> containers (Running game servers, e.g. tf2)

Wireguard is used to tunnel this traffic. 

If you can find a VPS close to your production server, the latency difference is barely noticable.

There is probably a more efficient way to do this.

Install instructions written for Ubuntu. 

Assumes you have OS firewall enabled (e.g. `ufw enable`, watch out and don't lock yourself out of SSH! you need to allow port 22)

## Installation
### 1. Installing wireguard and setting up the tunnel
1. Install wireguard `sudo apt install wireguard` on both servers (Tunneling and Pterodactyl)
2. Create the wireguard network interface `ip link add dev wg0 type wireguard` on both servers
3. Tunneling server: download the 'tunneling-wg0.conf' file and put it into your '/etc/wireguard/' folder
4. Tunneling server: Run `wg setconf wg0 /etc/wireguard/tunneling-wg0.conf` to link the config file to your network interface
5. Tunneling server: Run `wg genkey | tee privatekey | wg pubkey > publickey` to generate your private and public keys for the network interface
6. Tunneling server: Run `cat privatekey` and `cat publickey`. Take note of these values somewhere, you'll need these to fill the configs later.
7. Pterodactyl server: download the 'pterodactyl-wg0.conf' file and put it into your '/etc/wireguard/' folder
8. Pterodactyl server: Run `wg setconf wg0 /etc/wireguard/pterodactyl-wg0.conf` to link the config file to your network interface
9. Pterodactyl server: Run `wg genkey | tee privatekey | wg pubkey > publickey` to generate your private and public keys for the network interface
10. Pterodactyl server: Run `cat privatekey` and `cat publickey`. Take note of these values somewhere, you'll need these to fill the configs later.
11. Tunnel Server: Run `ip a` and note the top network interface you're using. Should be something like `eth0` or `ens`. Should NOT be `loop` or something similar. If it is, pick the network interface below that one. You will need this for the config file.
12. Edit both .conf files with the appropriate values for your servers
13. Run `wg-quick up wg0` on both servers to activate the tunnel. If you get a "wg0 already exists" popup, run `wg-quick down wg0` and then `wg-quick up wg0` **WARNING: Always run `wg-quick down wg0` before editing a config file, otherwise you will cause changes which will require manual tinkering with iptables to revert!**
14. Continue to chapter 2.

### 2. Port forwarding & firewall
1. Tunneling and Pterodactyl server: Allow the required ports in your OS firewall (e.g. run `ufw allow 27000:27050/udp` and `ufw allow 27000:27050/tcp`
2. Tunneling server: Allow the Wireguard port in your OS firewall (e.g. run `ufw allow 51820`)
3. Tunneling server: If your tunneling server has an outside firewall (e.g. a router, your VPS service might have a firewall, etc) port forward ports 51820 and the range 27000 to 27050. Google how to do this, it's specific to your circumstances
4. Both servers: Edit your /etc/sysctl.conf file (e.g. run `sudo nano /etc/systemctl.conf`) and uncomment `#net.ipv4.ip_forward = 1` -> `net.ipv4.ip_forward = 1`. Save the file
5. Continue to 3.

### 3. Testing
14. Tunneling server: run `ping 10.0.0.2`. If everything was set up correctly, you should see packets being sent from your tunneling server to your pterodactyl server
15. Pterodactyl server: run `ping 10.0.0.1`. If everything was set up correctly, you should see packets being sent from your pterodactyl server to your tunneling server
16. Press Ctrl+C on both servers to cancel the pinging commands
17. Open tf2 and connect to your game server via your tunneling server ip, ``connect TunnelingServerIP:Port``
18. If you can connect, great! Move to step 4.

### 4. Autoload on reboot
To make sure the wireguard tunnels autostart when you reboot either server, run these commands:
1. Both servers: run `systemctl enable wg-quick@wg0`

Your game traffic should now be tunneled and you should be able to connect to your TF2 server by connecting to your tunnel server's IP:PORT .
