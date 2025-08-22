# gboexe
Esta herramienta nos permite realizar escaneos de manera mas eficiente, optimizando tiempo y recursos al obtener resultados mas completos y precisos.
#!/bin/bash

# Script de enumeración avanzada estilo Metasploit
# Autor: xHadar
# Uso: ./auto_enum_menu.sh <IP o URL>
# Requiere: nmap, gobuster, curl, nc

TARGET="$1"
WORDLIST="/usr/share/wordlists/dirb/common.txt"

# Colores
RED="\e[31m"
GREEN="\e[32m"
BLUE="\e[34m"
YELLOW="\e[33m"
PURPLE="\e[35m"
RESET="\e[0m"

if [ -z "$TARGET" ]; then
    echo -e "${RED}Uso: $0 <IP o URL>${RESET}"
    exit 1
fi

# ----------------------
# Banner estático
# ----------------------
clear
echo -e "${PURPLE}"
echo "   __  ___      __      __  _               "
echo "  /  |/  /___ _/ /___  / /_(_)___  ____ ___ "
echo " / /|_/ / __ \`/ / __ \/ __/ / __ \/ __ \`__ \\"
echo "/ /  / / /_/ / / /_/ / /_/ / / / / / / / /"
echo "/_/  /_/\__,_/_/\____/\__/_/_/ /_/_/ /_/ / "
echo -e "${GREEN}         Enumeración Avanzada v1.0${RESET}"
echo -e "${BLUE}             Autor: xHadar${RESET}"
echo -e "${YELLOW}==============================================${RESET}"

# ----------------------
# Menú de opciones
# ----------------------
while true; do
    echo -e "${BLUE}"
    echo "Seleccione el tipo de escaneo a realizar:"
    echo "1) Escaneo de puertos comunes"
    echo "2) Escaneo web con Gobuster"
    echo "3) Detección de encabezados HTTP"
    echo "4) Salir"
    echo -e "${RESET}"
    read -p "Ingrese una opción [1-4]: " OPTION

    case $OPTION in
        1)
            echo -e "${YELLOW}Escaneando puertos comunes (21,22,80,443,8080)...${RESET}"
            nmap -p 21,22,80,443,8080 -sV -oN nmap_services_$TARGET.txt $TARGET
            echo -e "${GREEN}[+] Escaneo completado. Resultados en nmap_services_$TARGET.txt${RESET}"
            ;;
        2)
            echo -e "${YELLOW}Escaneo web con Gobuster...${RESET}"
            for PORT in 80 443 8080; do
                nc -z -w 3 $TARGET $PORT &> /dev/null
                if [ $? -eq 0 ]; then
                    URL="http://$TARGET"
                    [[ $PORT -eq 443 ]] && URL="https://$TARGET"
                    [[ $PORT -eq 8080 ]] && URL="http://$TARGET:8080"
                    echo -e "${GREEN}[+] Puerto $PORT abierto, ejecutando Gobuster en $URL...${RESET}"
                    gobuster dir -u $URL -w $WORDLIST -o gobuster_${PORT}_$TARGET.txt
                    echo -e "${GREEN}[+] Gobuster completado en puerto $PORT${RESET}"
                else
                    echo -e "${RED}[-] Puerto $PORT cerrado o filtrado${RESET}"
                fi
            done
            ;;
        3)
            echo -e "${YELLOW}Detectando encabezados HTTP...${RESET}"
            curl -s -I http://$TARGET 2>/dev/null | grep -i "server\|x-powered-by" > headers_$TARGET.txt
            curl -s -I https://$TARGET 2>/dev/null | grep -i "server\|x-powered-by" >> headers_$TARGET.txt
            echo -e "${GREEN}[+] Encabezados guardados en headers_$TARGET.txt${RESET}"
            ;;
        4)
            echo -e "${BLUE}Saliendo...${RESET}"
            exit 0
            ;;
        *)
            echo -e "${RED}Opción inválida, intente de nuevo.${RESET}"
            ;;
    esac
done
