# R5Reloaded Dedicated For Pufferpanel

This tutorial will show you how to get pufferpanel up and running including adding the r5r dedi template to it.

This was installed using Ubuntu 20.04 as the OS. <br>
Others should work but have not tested them.

### Installing PufferPanel using Docker
PufferPanel offers several images that include dependencies needed to run game servers. We recommend using latest as it contains everything you will need to get servers runing quickly.

### Creating the container
To create the container, start it, and add the default user:

```
mkdir -p /var/lib/pufferpanel
docker volume create pufferpanel-config
docker create --name pufferpanel -p 8080:8080 -p 5657:5657 -p 37015-37020:37015-37020/tcp -p 37015-37020:37015-37020/udp -v pufferpanel-config:/etc/pufferpanel -v /var/lib/pufferpanel:/var/lib/pufferpanel --restart=on-failure pufferpanel/pufferpanel:latest
pufferpanel/pufferpanel:latest
docker start pufferpanel
docker exec -it pufferpanel /pufferpanel/pufferpanel user add
```

And you’re done. Your panel is now accessible at http://localhost:8080

### Adding the R5Reloaded Dedicated Template
To add the Template you will want to login to your panel and click on Templates

![chrome_OD9Y6HUcvx](https://user-images.githubusercontent.com/18438498/166391531-70311e8a-e47f-47e8-a6b8-efd5d596532b.png)

Click on the + Button

![chrome_OD9Y6HUcvx](https://user-images.githubusercontent.com/18438498/166391591-9eadbf3d-2159-4966-bd34-2370dc75886d.png)

Switch over to JSON view in the top right


![chrome_BO7vG80Evv](https://user-images.githubusercontent.com/18438498/166391650-5810ae08-b8c2-470f-bb72-e99bec7c30b2.png)

Replace all the JSON shown with this json
```
{
  "name": "R5Reloaded Dedicated",
  "display": "R5Reloaded Dedicated",
  "type": "r5r",
  "install": [
    {
      "commands": [
        "apt-get update",
        "apt-get install unzip",
        "apt-get install wine64"
      ],
      "type": "command"
    },
    {
      "files": [
        "https://downloads.r5reloaded.com/builds/R5Dedicated.zip",
        "https://github.com/Mauler125/r5sdk/releases/download/v2.0.6_rc1/depot.zip",
        "https://github.com/Mauler125/scripts_r5/archive/refs/heads/S3_N1094.zip",
        "https://downloads.r5reloaded.com/DediFix.zip"
      ],
      "type": "download"
    },
    {
      "commands": [
        "unzip ./R5Dedicated.zip",
        "unzip -o ./depot.zip",
        "unzip -o ./scripts_r5-S3_N1094.zip",
        "mv ./scripts_r5-S3_N1094/ ./platform/scripts/",
        "unzip -o ./DediFix.zip",
        "rm -r ./platform/cfg/autoexec.cfg",
        "touch ./platform/cfg/autoexec.cfg"
      ],
      "type": "command"
    },
    {
      "target": "./platform/cfg/autoexec.cfg",
      "text": "//////////////////////////\n//// REPLICATED       ////\n//////////////////////////\nsv_cheats                    \"0\" // Enables cheats on the server.\nmp_allowed                   \"1\" // Whether multiplayer is allowed or not.\ndeveloper                    \"1\" // Required for certain script functionality.\n\n//////////////////////////\n//// NETCHAN          ////\n//////////////////////////\nnet_userandomkey             \"1\" // Whether to use the default netkey or a randomized key on launch.\nnet_usesocketsforloopback    \"1\" // Whether to use network sockets layer for packets.\n\n//////////////////////////\n//// SQUIRREL         ////\n//////////////////////////\nsq_showrsonloading           \"1\" // Shows the global include files the SQVM is loading for loading scripts.\nsq_showscriptloading         \"0\" // Shows the script files the SQVM is loading for precompile job.\nsq_showvmoutput              \"1\" // Shows the VM output. 1 = Log to file. 2 = 1 + log to console. 3 = 1 + 2 + log to overhead console. 4 = only log to overhead console.\nsq_showvmwarning             \"1\" // Shows the VM warning output. 1 = Log to file. 2 = 1 + log to console.\n\nexec serverconfig",
      "type": "writefile"
    },
    {
      "commands": [
        "rm -r ./R5Dedicated.zip",
        "rm -r ./depot.zip",
        "rm -r ./scripts_r5-S3_N1094.zip"
      ],
      "type": "command"
    }
  ],
  "run": {
    "stop": "wineserver -k",
    "command": "./startserver.sh",
    "workingDirectory": "",
    "pre": [
      {
        "commands": [
          "rm -r ./platform/cfg/serverconfig.cfg",
          "touch ./platform/cfg/serverconfig.cfg",
          "rm -r ./platform/cfg/rcon_server.cfg",
          "touch ./platform/cfg/rcon_server.cfg"
        ],
        "type": "command"
      },
      {
        "target": "./platform/cfg/serverconfig.cfg",
        "text": "port \"${Port}\"\nhostport \"${Port}\"\nhostname \"${HostName}\"\nsv_pylonvisibility \"${Visibility}\"\nlaunchplaylist \"${Playlist}\"\nsv_cheats \"${sv_cheats}\"\nsv_requireOriginToken \"${sv_requireOriginToken}\"\nr5net_show_debug \"${r5net_debug_enabled}\"\n",
        "type": "writefile"
      },
      {
        "target": "./platform/cfg/rcon_server.cfg",
        "text": "////////////////// rcon_server configuration file.\n// This file is executed automatically on startup.\n// See https://developer.valvesoftware.com/wiki/Source_RCON_Protocol for more information regarding RCON.\n// NOTE: This implementation is custom and differs slightly from Valve's implementation.\n\nrcon_password             \"${rcon_password}\"                 // !! WARNING !! Keep empty to disable RCON. Only enable this if you plan on using RCON.\nsv_rcon_whitelist_address \"${sv_rcon_whitelist_address}\" // This IP will never get disconnected or banned. Enter your own IP address in the same format here (counts for IPv4 and IPv6).\n",
        "type": "writefile"
      }
    ],
    "post": [
      {
        "commands": [
          "wineserver -k"
        ],
        "type": "command"
      }
    ],
    "environmentVars": {}
  },
  "data": {
    "HostName": {
      "type": "string",
      "desc": "Server Name",
      "display": "HostName",
      "value": "R5R Dedicated Server"
    },
    "Playlist": {
      "type": "string",
      "desc": "Set the gamemode loaded on start",
      "display": "Playlist",
      "required": true,
      "value": "custom_tdm",
      "userEdit": true
    },
    "Port": {
      "type": "integer",
      "desc": "Set server port",
      "display": "Port",
      "required": true,
      "value": "37015",
      "userEdit": true
    },
    "Visibility": {
      "type": "integer",
      "desc": "Is the server visible on the server list. ( 1 for visible )",
      "display": "Server Visibility",
      "value": "1"
    },
    "r5net_debug_enabled": {
      "type": "integer",
      "desc": "Shows post info for server browser ( will cause lag when server has been up for awhile )",
      "display": "r5net_debug_enabled",
      "required": true,
      "value": "0",
      "userEdit": true
    },
    "rcon_password": {
      "type": "string",
      "desc": "Keep empty to disable RCON. Only enable this if you plan on using RCON.",
      "display": "Rcon Password",
      "userEdit": true
    },
    "sv_cheats": {
      "type": "integer",
      "desc": "Enables cheats on the server",
      "display": "sv_cheats",
      "required": true,
      "value": "0",
      "userEdit": true
    },
    "sv_rcon_whitelist_address": {
      "type": "string",
      "desc": "This IP will never get disconnected or banned. Enter your own IP address in the same format here (counts for IPv4 and IPv6).",
      "display": "Rcon Whitelist Address",
      "userEdit": true
    },
    "sv_requireOriginToken": {
      "type": "integer",
      "desc": "Enables origin token verification on the server",
      "display": "sv_requireOriginToken",
      "required": true,
      "value": "0",
      "userEdit": true
    }
  },
  "environment": {
    "type": "tty"
  },
  "supportedEnvironments": [
    {
      "type": "tty"
    }
  ]
}
```

Click save and your good to go!
