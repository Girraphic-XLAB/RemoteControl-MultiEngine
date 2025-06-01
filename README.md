# Unreal RC Web Application Connection Replication

By default, Unreal Engine’s Remote Control web application plugin sends Websocket requests to a single IP address. Our company required a single host machine replicating requests to multiple client machines for broadcast.

To achieve this, the source code for the web application plugin was edited to read a file from the host’s local storage with several IPs delimited by new lines. This list of IPs was used to create additional Websocket connections for each machine intended to be managed by the host.

Changes were also made to the websocket message send function to duplicate the websocket message to the initial host to each client connection. The following pages contain code examples of the before and after for each change.

### Connection Before

```
function connect() {
 if (connection?.readyState === WebSocket.OPEN || connection?.readyState === WebSocket.CONNECTING)
   return;


 const address = `ws://127.0.0.1:${Program.ueWebSocketPort}`;


 connection?.removeAllListeners();


 connection = new WebSocket(address);
 connection
   .on('open', onConnected)
   .on('message', onMessage)
   .on('error', onError)
   .on('close', onClose);
}
```

### Connection After
```

function connect() {
 if (connection?.readyState === WebSocket.OPEN || connection?.readyState === WebSocket.CONNECTING)
   return;


 const address = `ws://127.0.0.1:${Program.ueWebSocketPort}`;


 connection?.removeAllListeners();


 connection = new WebSocket(address);
 connection
   .on('open', onConnected)
   .on('message', onMessage)
   .on('error', onError)
   .on('close', onClose);
 //Just copy everything below this with the IP you want
 ips.forEach((value, index) => {
   const customAddress = `ws://${value}:${Program.ueWebSocketPort}`;


   connections[index] = new WebSocket(customAddress);
   connections[index]
       .on('open', onConnected)
       .on('message', onMessage)
       .on('error', onError)
       .on('close', onClose);
 });
}
```
### Send Before

```
function send(message: string, parameters: any, passphrase?: string, ip?: string) {
 verifyConnection();
 const Id = parameters?.RequestId ?? wsRequest++;
 passphrase = passphrase ?? workingPassphrases[0];
ip = ip ?? "";
 connection.send(JSON.stringify({ MessageName: message, Id, Passphrase: passphrase, ForwardedFor: ip, Parameters: parameters}));
}
```

### Send After

```
function send(message: string, parameters: any, passphrase?: string, ip?: string) {
 verifyConnection();
 const Id = parameters?.RequestId ?? wsRequest++;
 passphrase = passphrase ?? workingPassphrases[0];
ip = ip ?? "";
 connection.send(JSON.stringify({ MessageName: message, Id, Passphrase: passphrase, ForwardedFor: ip, Parameters: parameters}));
 //Another one of these per connection
 connections.forEach(conn => {
   conn.send(JSON.stringify({ MessageName: message, Id, Passphrase: passphrase, ForwardedFor: ip, Parameters: parameters}));
 });


}
```
