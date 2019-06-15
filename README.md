# Shoukaku
<p align="center">
  <img src="https://vignette.wikia.nocookie.net/kancolle/images/9/97/Shoukaku_Christmas_Full.png/revision/latest/">
</p>
The ShipGirl Project. Shoukaku `(c) Kancolle for Shoukaku`

### A Full Blown Lavalink Wrapper designed around Discord.js v12

### Saya Note:
> The wrapper has entered the `beta` stage. 

>If you want to help in development, you can use the wrapper and report the issues you experienced on using it, or Submit a PR if you think you can improve something.

> Documentation is not yet available as of now, but I will soon:tm:

> Look at the Example Usage for an idea on what to see in this library

### Installation
```
npm i Deivu/Shoukaku
```
### Task List
- [x] Base Logic
- [x] Playing Stopping Logic 
- [x] Load Balancing Logic
- [ ] Node Removal logic
- [ ] Reconnect logic
- [ ] Resuming Logic
- [ ] Documentation
- and probably some things I forgot to mention?

### Example Usage
```js
const { Client } = require('discord.js');
const { Shoukaku } = require('shoukaku');
const client = new Client();

// In this example, I will assign Shoukaku to a carrier variable. Options are the default options if nothing is specified
const Carrier = new Shoukaku(client, {
  resumable: false,
  resumableTimeout: 30,
  reconnectTries: 2,
  restTimeout: 10000 
});
// Attach listeners, currently this are the only listeners available
// on ERROR must be handled.
Carrier.on('ready', (name) => console.log(`Lavalink Node: ${name} is now connected`));
Carrier.on('error', (name, error) => console.log(`Lavalink Node: ${name} emitted an error.`, error));
Carrier.on('close', (name, code, reason) => console.log(`Lavalink Node: ${name} closed with code ${code}. Reason: ${reason || 'No reason'}`));

client.on('ready', () => {
  // You need to build shoukaku on your client's ready event for her to work like how its done in this example.
  Carrier.build([{
    name: 'my_lavalink_server',
    host: 'localhost',
    port: 6969,
    auth: 'I_Love_Anime_Weeb_69'
  }], { 
    id: client.user.id 
  });
  console.log('Bot Initialized');
})

// Now I will show you how to make a simple handler that plays a link on your chnanel. Async Await style
client.on('message', async (msg) => {
  if (msg.author.bot || !msg.guild) return
  if (msg.content.startsWith('$play')) {
    // Check if there is already a link on your guild.
    if (Carrier.getLink(msg.guild.id)) return;
    const args = msg.content.split(' ');
    if (!args[1]) return;
    // Getting the recommended node, the ez way.
    const node = Carrier.getNode();
    // Getting the track data 
    let data = await node.rest.resolve(args[1]);
    if (!data) return;
    if (Array.isArray(data)) data = data[0];
    // Joining the Voice Channel to obtain a link class for the guild.
    const link = await node.joinVoiceChannel({
      guild_id: msg.guild.id,
      channel_id: msg.member.voice.channelID
    });
    // Attach the listeners. More would be added probably
    link.player.on('TrackEnd', () => {
      link.disconnect();
    })
    link.player.on('TrackException', console.log);
    link.player.on('TrackStuck', () => {
      link.player.stopTrack().catch(console.error);
    });
    link.player.on('WebSocketClosed', () => link.disconnect());
    // Do the play thingy
    await link.player.playTrack(data.track);
    await msg.channel.send("Now Playing: " + data.info.title);
  }
})
client.login('token');
```