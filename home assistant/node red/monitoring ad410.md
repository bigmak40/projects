# Monitoring the AD410

The Amcrest AD410 (rebranded Dahua) is a decent camera but constantly pings their home servers. After blocking it from internet access via firewall rules, the camera will continue to stream RTSP just fine but a timeout will eventually occur where it detects it does not have internet access. In order to alert the user, it will then start flashing green which is pretty annoying.

To prevent this from happening, I use a Node-RED flow that runs periodically. I'll step through each step in the flow and how it's configured. I know this might be dumbed down, but want to make sure anyone who only has a cursory knowledge of Node-RED can follow along.

## Backing up camera configuration

Before doing anything, I recommend querying the camera and backing up the full configuration. This way, in case you mess anything up you can see how it was originally configured. Doing so is easy; just go to the following URL and log in using your camera username and password (my password is admin, don't remember when this gets set).

`http://[camera local IP]/cgi-bin/configManager.cgi?action=getConfig&name=All`

## Automatically checking and fixing offline status

![Screenshot of flow](/01 overall flow.png)

1. Inject Node
   * Set to repeat every 15 minutes; probably can run less often, but this is fine
2. HTTP Request Node
   * Method: `POST`
   * URL: `http://[camera local IP]/cgi-bin/configManager.cgi?action=getConfig&name=VSP_PaaS.Online`
   * Enable Secure Connection: `disabled` (not checked)
   * Use Authentication: `enabled`
     * Type: `digest authentication`
     * Username: `username to log into camera`
     * Password: `password to log into camera`
   * Enable connection keep alive: `disabled`
   * Use proxy: `disabled`
   * Only send Non-2xx responses: `disabled`
   * Return: `a UTF-8 string`
3. Switch node
   * Property: `msg.payload`
   * Output #1:
     * Type: `contains`
     * String: `table.VSP_PaaS.Online=true`
   * Output #2:
     * Type: `contains`
     * String: `table.VSP_PaaS.Online=false`
4. HTTP Request Node (connected to output #2 from node #3)
   * *The only difference compared to node #2 is that this is a setConfig and the value is listed*
   * Method: `POST`
   * URL: `http://[camera local IP]/cgi-bin/configManager.cgi?action=setConfig&VSP_PaaS.Online=true`
   * Enable Secure Connection: `disabled` (not checked)
   * Use Authentication: `enabled`
     * Type: `digest authentication`
     * Username: `username to log into camera`
     * Password: `password to log into camera`
   * Enable connection keep alive: `disabled`
   * Use proxy: `disabled`
   * Only send Non-2xx responses: `disabled`
   * Return: `a UTF-8 string`
5. This is a node I made which logs things to a debug Telegram channel, but does not effect how this works.
