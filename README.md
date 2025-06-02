# Unreal RC Web Interface Replication

The native Unreal Engine’s RC Web Interface plugin sends Websocket requests to a single IP address. The `UnrealEngine.ts` file in this project allows a single host machine to replicate requests to multiple client machines.

The web interface plugin file at `..\UE5.3\Engine\Plugins\VirtualProduction\RemoteControlWebInterface\WebApp\Server\src` can be edited to read a file from the host’s file system [`..\UE5.3\Engine\Plugins\VirtualProduction\RemoteControl\WebInterface\WebApp\ip.txt`] with each IP delimited by new lines. For each IP in the list, a websocket connection is established to the host's web interface.

<table>
<tr>
<th>Before</th>
<th>After</th>
</tr>
<tr>
<td>
<pre>
 
```typescript
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

</pre>
</td>
<td>

```typescript
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

</td>
</tr>
</table>

 The websocket message send function was changed to duplicate the message across client connections.

<table>
<tr>
<th>Before</th>
<th>After</th>
</tr>
<tr>
<td>
<pre>
 
```typescript
function send(message: string, parameters: any, passphrase?: string, ip?: string) {
 verifyConnection();
 const Id = parameters?.RequestId ?? wsRequest++;
 passphrase = passphrase ?? workingPassphrases[0];
ip = ip ?? "";
 connection.send(JSON.stringify({ MessageName: message, Id, Passphrase: passphrase, ForwardedFor: ip, Parameters: parameters}));
}
```

</pre>
</td>
<td>

```typescript
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

</td>
</tr>
</table>
