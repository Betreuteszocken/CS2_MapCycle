<h1>🗺️ CS2_MapCycle – Automatic map rotation for CS2</h1>
<p>🇬🇧 English | 🇩🇪 <a href="README.de.md">Deutsch</a></p>
<p>
<strong>CS2_MapCycle</strong> is a CounterStrikeSharp plugin for Counter-Strike 2.<br>
When a match ends, it automatically switches to the next map from your own rotation, either in order or at random.
The built-in CS2 map vote is ignored.
</p>
<ul>
  <li>Custom mapcycle file with regular maps and Workshop maps</li>
  <li>Sequential or random order, optionally without repeats</li>
  <li>Chat announcement of the next map at the start of the last round</li>
  <li>Reads <code>mp_maxrounds</code> live, even if the value changes during a match</li>
  <li>Configurable chat prefix (e.g. your server name)</li>
</ul>
<hr>
<h2>📦 Requirements</h2>
<ul>
  <li>CS2 Dedicated Server</li>
  <li><a href="https://github.com/roflmuffin/CounterStrikeSharp">CounterStrikeSharp</a> v1.0.380 or newer (with Metamod:Source)</li>
</ul>
<h2>🔧 Installation</h2>
<ol>
  <li>Copy the plugin folder to <code>game/csgo/addons/counterstrikesharp/plugins/CS2_MapCycle/</code>.</li>
  <li>Start the server. The config file is created automatically on first start.</li>
  <li>Edit the mapcycle file <code>game/csgo/mapcyclecustom.txt</code> (see below).</li>
</ol>
<hr>
<h2>⚙️ Configuration</h2>
<p>File: <code>game/csgo/addons/counterstrikesharp/configs/plugins/CS2_MapCycle/CS2_MapCycle.json</code></p>
<pre><code>{
  "ConfigVersion": 2,
  "PluginEnabled": true,
  "EnableRandomMaps": false,
  "EnableNoDuplicateRandomMaps": true,
  "EnableNextMapMessage": true,
  "ChatPrefix": "[MapCycle]",
  "MapCycleFile": "mapcyclecustom.txt"
}
</code></pre>
<table>
  <thead>
    <tr><th>Option</th><th>Default</th><th>Description</th></tr>
  </thead>
  <tbody>
    <tr><td><code>PluginEnabled</code></td><td><code>true</code></td><td>Enable/disable the plugin</td></tr>
    <tr><td><code>EnableRandomMaps</code></td><td><code>false</code></td><td>Pick the next map at random instead of in order</td></tr>
    <tr><td><code>EnableNoDuplicateRandomMaps</code></td><td><code>true</code></td><td>In random mode, play every map once before any map repeats</td></tr>
    <tr><td><code>EnableNextMapMessage</code></td><td><code>true</code></td><td>Announce the next map in chat during the last round and when the match ends</td></tr>
    <tr><td><code>ChatPrefix</code></td><td><code>[MapCycle]</code></td><td>Text shown in front of every chat announcement, e.g. your server name. Leave empty (<code>""</code>) for no prefix</td></tr>
    <tr><td><code>MapCycleFile</code></td><td><code>mapcyclecustom.txt</code></td><td>Name of the mapcycle file in the <code>game/csgo/</code> folder</td></tr>
  </tbody>
</table>
<p>
ℹ️ <strong>Updating from an older version:</strong> CounterStrikeSharp does not add new options to an existing config file.
Add <code>"ChatPrefix"</code> manually and set <code>"ConfigVersion"</code> to <code>2</code>, or delete the file so a new one is created.
</p>
<hr>
<h2>🗒️ Mapcycle file</h2>
<p>
One map per line. Lines starting with <code>//</code> are comments.<br>
Workshop maps are entered as <code>WorkshopID:DisplayName</code>. The display name is shown in the chat announcement, with upper and lower case exactly as written in the file.
</p>
<pre><code>// Regular maps
de_dust2
de_inferno
de_mirage

// Workshop maps (ID:Name)
3070923343:fy_pool_day
</code></pre>
<p>
If the file is missing and is named <code>mapcyclecustom.txt</code>, the plugin creates an example file automatically.<br>
With a custom file name, <strong>nothing</strong> is created automatically. The plugin then reports an error in the console.
</p>
<hr>
<h2>⏱️ Timing after the match ends</h2>
<p>After the last round, two server settings decide when the map changes:</p>
<ul>
  <li><code>mp_win_panel_display_time</code>: how long the win/loss panel is shown</li>
  <li><code>mp_match_restart_delay</code>: how long CS2 waits before restarting or changing the map</li>
</ul>
<p>
The plugin takes the <strong>larger</strong> of the two values and changes the map <strong>5 seconds earlier</strong> (immediately at the earliest).<br>
Example: <code>mp_win_panel_display_time 3</code> and <code>mp_match_restart_delay 15</code> → map change after 10 seconds.
</p>
<p>
This way the plugin changes the map before CS2 steps in itself.
If CS2 tries to change the map on its own after the delay and <code>mp_endmatch_votenextmap 0</code> is set, the game may try to load an empty map. This can freeze the server.
</p>
<h3>✅ Recommended settings</h3>
<pre><code>mp_win_panel_display_time 3
mp_match_restart_delay 15      // enough time for the "next map" announcement
mp_endmatch_votenextmap 1      // keep enabled as a fallback, the vote is ignored
</code></pre>
<hr>
<h2>🤖 Bot-only servers</h2>
<p>
For the map to change when only bots are playing, two things must be set up correctly:
</p>
<h3>1. No hibernation</h3>
<pre><code>sv_hibernate_when_empty 0
</code></pre>
<p>While hibernating, no game ticks run, so no plugin timers run either.</p>
<h3>2. No warmup without real players</h3>
<p>
Without a real player, the CS2 warmup timer never runs out, even if <code>mp_warmuptime</code> is set.
The bots then play warmup forever, the match never ends, and the map never changes.
Only when a player joins does everything continue.
</p>
<p>Solution: disable warmup for online matches.</p>
<pre><code>mp_warmup_online_enabled false
</code></pre>
<p>
⚠️ <strong>Important:</strong> This line belongs in <code>gamemode_&lt;mode&gt;_server.cfg</code> (e.g. <code>gamemode_casual_server.cfg</code> for <code>game_type 0</code> / <code>game_mode 0</code>).
CS2 loads the game mode config on every map start, which overrides values from the regular server config.
</p>
<p>
Note: <code>mp_do_warmup_period</code> from CS:GO no longer exists in CS2.
To see which warmup commands your server supports, use the console command <code>find warmup</code>.
</p>
<hr>
<h2>📊 Summary</h2>
<table>
  <thead>
    <tr><th>Setting</th><th>Description</th><th>Recommended</th></tr>
  </thead>
  <tbody>
    <tr><td><code>mp_win_panel_display_time</code></td><td>Duration of the win/loss panel</td><td>3</td></tr>
    <tr><td><code>mp_match_restart_delay</code></td><td>Delay after the match ends</td><td>15</td></tr>
    <tr><td><code>mp_endmatch_votenextmap</code></td><td>CS2 map vote (ignored by the plugin)</td><td>1</td></tr>
    <tr><td><code>sv_hibernate_when_empty</code></td><td>Hibernation when the server is empty</td><td>0</td></tr>
    <tr><td><code>mp_warmup_online_enabled</code></td><td>Warmup in online matches</td><td>false</td></tr>
  </tbody>
</table>
<hr>
<h2>📝 License</h2>
<p>See <code>LICENSE</code>.</p>
