Enterprise Attack Simulation & Defensive Monitoring Lab
1. Introdução
Este projeto apresenta a construção de um laboratório de simulação de ataques e monitoramento defensivo com o objetivo de demonstrar técnicas de detecção e resposta a incidentes utilizando o SIEM Wazuh. O ambiente foi projetado para simular ataques comuns encontrados em redes corporativas e demonstrar como ferramentas de segurança podem detectar, correlacionar e responder automaticamente a atividades maliciosas.
2. Arquitetura do Laboratório
A infraestrutura é composta pelos seguintes componentes:

Sistema Atacante: Kali Linux.
Sistema Alvo: Ubuntu Server.
Plataforma SIEM: Wazuh.
Serviço Monitorado: OpenSSH.

3. Simulação de Ataque (Red Team)
Nesta etapa, foi realizado um ataque de força bruta contra o serviço SSH utilizando a ferramenta Hydra.

Comando utilizado: hydra -l root -P rockyou.txt ssh://192.168.1.106.

O ataque gera múltiplas tentativas de autenticação.

As tentativas de login foram registradas no arquivo de log do sistema: /var/log/auth.log

4. Detecção e Monitoramento (Blue Team)
Os logs gerados no servidor alvo foram coletados automaticamente pelo agente do Wazuh. O SIEM analisou os logs e detectou padrões de comportamento suspeitos através de regras personalizadas.

Regras Personalizadas (local_rules.xml)Foram implementadas regras para identificar o ataque:Regra 100200: Detecta uma falha de login individual.Regra 100201: Detecta o ataque de força bruta (frequência de 5 tentativas em 60 segundos).Mapeamento MITRE ATT&CK: Técnica T1110 (Brute Force).

5. Resposta Automática (Active Response)
Foi configurado o recurso Active Response do Wazuh para bloquear automaticamente o IP atacante.
Ação: Execução do script firewall-drop.

Configuração de Bloqueio: O IP é bloqueado por um tempo determinado (timeout) de 180 segundos.

Implementação Técnica: O bloqueio foi aplicado no firewall do sistema utilizando nftables.

Resultado: Após o bloqueio, novas conexões SSH foram impedidas, resultando em Connection timed out para o atacante.

6. Resultados e Visibilidade
O dashboard do Wazuh fornece visibilidade centralizada, permitindo identificar padrões de ataque e verificar as respostas automáticas em tempo real. Este laboratório demonstrou com sucesso:

Detecção de ataques SSH.
Correlação de eventos e integração com MITRE ATT&CK.
Resposta automática com bloqueio de IP.