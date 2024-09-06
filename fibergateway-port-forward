#!/bin/bash

checkDependencies() {
    dependencies=("dialog" "sshpass")

    for dep in "${dependencies[@]}"; do
        if [ ! "$(command -v "$dep")" ]; then
            fatal "[ERROR] Command $dep not installed."
        fi
    done

    if [ $EUID != 0 ]; then
        fatal "[ERROR] Sudo privileges required."
    fi
}


checkLogin() {
    sshpass -p "$routerPass" ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no "$routerUser"@"$routerIP" &>/dev/null <<-EOF 
	EOF
}

checkVariables() {
    variables=("routerIP" "routerUser" "routerPass")
    variables+=("$@")

    for var in "${variables[@]}"; do
        if [ -z "${!var}" ]; then
            fatal "[ERROR] Variable $var not set."
        fi
    done
}

fatal() {
    echo "$@" >&2
    kill -10 $proc
}

remoteExec() {
    rules=("$@")

    execOutput=$(sshpass -p "$routerPass" ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no "$routerUser"@"$routerIP" 2>/dev/null <<-EOF
		$(for rule in "${rules[@]}"; do
			echo "$rule"
		done)
		EOF
    )

    if [ -n "$execOutput" ]; then
        echo "$execOutput"
    else
        echo "[ERROR] Something went wrong with sshpass."
    fi
}

proc=$$
routerIP=
routerUser=
routerPass=
jellyfinIP=
playstation5IP=

trap 'exit 1' SIGUSR1

checkDependencies

if checkLogin; then
    while true; do
        menu=$(dialog --clear \
            --title "MEO FiberGateway Port Forwarding" \
            --menu "Select one of the following actions:" \
            0 0 0  \
            1 "Show Rules" \
            2 "Add Rule: Jellyfin" \
            3 "Remove Rule: Jellyfin" \
            4 "Add Rule: PS5 Remote Play" \
            5 "Remove Rule: PS5 Remote Play" \
            6 "Exit" \
            2>&1 >"$(tty)")
    
        clear
    
        case $menu in
            1)  
                checkVariables 
                dialog --msgbox "$(remoteExec "/nat/virtual-servers/show" | tail -n +11 | head -n -3)" 0 0
                ;;
            2)
                checkVariables "jellyfinIP"
                dialog --msgbox "$(remoteExec "/nat/virtual-servers/create --server-name=Jellyfin --server-ip=$jellyfinIP --protocol=TCP --ext-port-start=8096 --int-port-start=8096 --wan-intf=erouter0" | tail -n +11 | head -n -4 | sed 's/SUCCESS//g')" 0 0
                ;;
            3)
                checkVariables "jellyfinIP"
                dialog --msgbox "$(remoteExec "/nat/virtual-servers/remove --server-ip=$jellyfinIP --protocol=TCP --ext-port-start=8096 --int-port-start=8096" | tail -n +11 | head -n 2)" 0 0
                ;;
            4)
                checkVariables "playstation5IP"
                dialog --msgbox "$(remoteExec "/nat/virtual-servers/create --server-name=PlayStation5 --server-ip=$playstation5IP --protocol=UDP --ext-port-start=9302 --int-port-start=9302 --wan-intf=erouter0" "/nat/virtual-servers/create --server-name=PlayStation5 --server-ip=$playstation5IP --protocol=TCP --ext-port-start=9295 --int-port-start=9295 --wan-intf=erouter0" "/nat/virtual-servers/create --server-name=PlayStation5 --server-ip=$playstation5IP --protocol=UDP --ext-port-start=9295 --ext-port-end=9297 --int-port-start=9295 --int-port-end=9297 --wan-intf=erouter0" | tail -n +11 | head -n -4 | sed -e '10,13d;23,26d' -e 's/SUCCESS//g' | grep -Ev "command|wan-intf")" 0 0
                ;;
            5)
                checkVariables "playstation5IP"
                dialog --msgbox "$(remoteExec "/nat/virtual-servers/remove --server-ip=$playstation5IP --protocol=UDP --ext-port-start=9302 --int-port-start=9302" "/nat/virtual-servers/remove --server-ip=$playstation5IP --protocol=TCP --ext-port-start=9295 --int-port-start=9295" "/nat/virtual-servers/remove --server-ip=$playstation5IP --protocol=UDP --ext-port-start=9295 --ext-port-end=9297 --int-port-start=9295 --int-port-end=9297" | tail -n +11 | head -n -4 | sed '2,3d;6,7d')" 0 0
                ;;
            6)
                exit
                ;;
            *)
                exit
                ;;
        esac
    done
else
    fatal "[ERROR] Invalid username or password."
fi
