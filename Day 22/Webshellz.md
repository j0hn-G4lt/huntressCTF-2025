## Flag 1: `flag{03638631595684f0c8c461c24b0879e6}`

• Sysmon logs showing `IIS_USER` account creation
• Encoded string: `VJGSuERc6qYAYPdRc556JTHqxqWwLbPwzABc0XgIhgwYEWdQji1`
• Decode the `IIS_USER` password from base62

## Flag 2: `flag{c7ba76c0a4484fe8c135a1195e8d94ed}`

• Uploaded `frpc.ini` configuration file in HTTP objects
• Encoded string: `MZWGCZ33MM3WEYJXGZRTAYJUGQ4DIZTFHBRTCMZVMEYTCOJVMU4GIOJUMVSH2===`
• Extract strings from `revshell(34).aspx`, find the base32 string in the `server_port comment` and decode

## Flag 3: `flag{fb4e078a739ac4ce687eb78c2e51aafe}`

• PCAP traffic, HTTP POST request in frame 502
• Encoded string: `ZmxhZ3tmYjRlMDc4YTczOWFjNGNlNjg3ZWI3OGMyZTUxYWFmZX0=`
• Filter PCAP for POST requests, extract the failed login attempt value, decode from base64
