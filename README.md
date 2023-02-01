# eBPF tcp_mon_block

This eBPF program uses netlink TC, kernel tracepoints and kprobes to monitor connections from given PIDs (usually HTTP web servers) and block outgoing connections to all addresses initiated from them, unless they were listed in allow_list.json 

To run the example:

    1. Run python3 web_server.py . Note the server's PID (will be printed to stdout)
    2. Add the server's PID to allow_list.json . You can replace the the first entry on the JSON file and put your PID instead
    3. Run python3 tcp_mon_block.py -i interface_name (-v for verbose output)
    4. Put your web_server's listening IP in 'server_address' variable in http_client.py and run python3 http_client.py 

**Explanation**:

web_server.py is a simple HTTP web server built with flask. It has a SSRF vulnerability in the route to /public_ip  (you can read more about this vulnerability here https://portswigger.net/web-security/ssrf).

This route demonstrates a web server which connects to some remote API server (which is pretty common behavior) and recevies some data. The attached POC simply connects to https://api.ipify.org and fetches the server's public IP, then sends it back to the client.  
However, this specific route receives the API address to connect to from the user (http_client.py is used as the client in this POC, but in real life scenarios this will probably be a web browser). 

This creates a SSRF vulnerability as an attacker can put any address he/she wishes to force the web server to connect to it instead of the intended API address (https://api.ipify.org)

Run the POC twice - first, run only the web_server.py and http_client.py . http_client.py will send 2 requests to the web server:

    - The first one send HTTP GET request to the web server with 'https://api.ipify.org' address as the 'api' parameter, as intended to be used by the web server.
    - The second one sends HTTP GET request to the web server with 'https://api.macvendors.com' address as the 'api' parameter. This exploits the vulnerability, as it forces the web server to connect to a different address than intended at /public_ip route.


Now run the POC again, but this time add the web server's PID to allow_list.json as mentioned earlier. 

**Prerequisites**: 

    1. BCC and pyroute2 for tcp_mon_block
    2. Python3 flask and reqeusts in order to run the web_server.py and http_client.py POC
    3. Tested on Ubuntu with kernel version 5.15.0-57


![alt text](https://github.com/agentzex/ebpf_tcp_mon_block/blob/main/screenshots/1.JPG)


After web server initiated connection to non-allowed address:

![alt text](https://github.com/agentzex/ebpf_tcp_mon_block/blob/main/screenshots/2.JPG)

