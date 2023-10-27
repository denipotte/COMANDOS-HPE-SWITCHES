# Guia de comandos Switchs e Roteadores HPE COMWARE 5

    # Acesso via terminal/console
        Instalar putty ou hyperterminal
        Configuração: Velocidade: 9600 || Bits de dados: 8 || Paridade: Nenhum || Bits de parada: 1 || Controle de fluxo: Nenhum
        Plugar cabo console no switch e computador

    # Configuração inicial

        >> Entrar em modo de configuração:
            system-view

        >> Mudar nome do Switch:
            sysname SW01

        >> Timezone
            clock timezone Brasil minus 03:00:00

        >> Criar usuário local:
            local-user admin

        >> Atribuir senha com criptografia:
            password cipher minha_senha

        >> Nível de acesso do usuário (Neste exemplo permite acesso nível 3 ou Total):
            authorization-attribute level 3
            service-type ssh terminal level 3

            *** Nos Switches novos ***
                authorization-attribute user-role network-admin
                authorization-attribute user-role network-operator
                service-type ssh terminal

        >> Ativar autenticação via SERIAL
            user-interface aux 0 1
            authentication-mode scheme

        >> Ativar autenticação ia terminal e liberar acesso por SSH:
            user-interface vty 0 4
            authentication-mode scheme
            protocol inbound ssh

        >> Liberar acesso do usuário no switch remoto:
            service-type ssh telnet terminal
            service-type web

        >> Configurar um IP para acesso ao Switch:
            interface vlan-interface130
            ip address 10.17.0.40 255.255.0.0


        ---------------------------------------------------MANAGER VLAN---------------------------------------------------------
        >> Tirar vlan management e excluir a vlan de gerencia dos switches 3COM:
            undo management-vlan
            undo interface Vlan-interface 1

        >> Colocar Vlan de gerencia
            vlan 410
            name FMRP_MANAGER

        >> Definir vlan de gerencia 
            manager vlan 410

 -------------------------------------------------------STACKING-------------------------------------------------------------------
                                    >> Intelligent Resilient Framework (IRF) <<
                                        Stacking (Empilhamento de Switches)  

        Obs.: Limite de 4 switches no empilhamento. Desplugue as portas IRF.         

        >> Verificar em qual SLOT está a interface do IRF:
            display current-configuration
        
        >> Configurando Nome e Domínio somente no Switch Master:
            system-viewsysname STI-FMRP
            irf domain 017000

               *** Obs.: Salve e reinicie o Switch ***
        
        >> Configurando as interfaces:
            interface Ten-gigabitEthernet1/1/1
            shutdown
            interface Ten-gigabitEthernet1/1/2
            shutdown
            irf-port 1/1
            port group interface Ten-gigabitEthernet1/1/1 mode normal
            irf-port 1/2
            port group interface Ten-gigabitEthernet1/1/2 mode normal
            interface Ten-GigabitEthernet1/1/1 undo shutdown
            interface Ten-GigabitEthernet1/1/2 undo shutdown

        >> Definindo prioridade dos Switches (Quanto maior o número, maior a prioridade):
            irf member 1 priority 32
            save
            irf-port-configuration active
            irf mac-address persistent timer
            irf auto-update enable
            undo irf link-delay
            save

        
        >> Por padrão os Switches vêm com a numeração 1 de pilha. Tem que renumerar.
            A partir do segundo Switch:
            irf member 1 renumber 2

        >> Assim sucessivamente para o 3º e 4º:
            irf member 1 renumber 3
            irf member 1 renumber 4

           *** Salve, plugue os cabos nas portas Tera e reinicie os Switches ***

        >> Verificando a estrutura
            display irf
            display irf topology

        >> Criando VLAN’s <<
            //tagged
            vlan 100
            SW01 – VLAN100> description WIRELESS
            SW01 – VLAN100> name WIRELESS

      
            //tagged
            vlan 200
            SW01 – VLAN200> description VOIP
            SW01 – VLAN200> name VOIP
            //untagged
            vlan 402
            SW01 – VLAN402> description FMRP
            SW01 – VLAN402> name FMRP
            //tagged
            vlan 410
            SW01 – VLAN200> description VLAN410
            SW01 – VLAN200> name VLAN410
            //tagged
            vlan 808
            SW01 – VLAN200> description CLOCK
            SW01 – VLAN200> name CLOCK

        >> Associando VLAN’s às portas: <<
            SW01> interface GigabitEthernet1/1/1
            SW01[ interface GigabitEthernet1/1/1]> port link-type trunk
            SW01[ interface GigabitEthernet1/1/1 ]> port trunk permit vlan 1 100 200 402

            SW01> interface GigabitEthernet1/1/2
            SW01[ interface GigabitEthernet1/1/2]> port link-type hybrid
            SW01[ interface GigabitEthernet1/1/2 ]> port hybrid vlan 200 tagged
            SW01[ interface GigabitEthernet1/1/2 ]> port hybrid vlan 402 untagged
            SW01[ interface GigabitEthernet1/1/2 ]> port hybrid pvid vlan 402

            SW01> interface GigabitEthernet1/1/10
            SW01[ interface GigabitEthernet1/1/1 ]> port link-type hybrid
            SW01[ interface GigabitEthernet1/1/1 ]> undo port hybrid vlan 1
            SW01[ interface GigabitEthernet1/1/1 ]> port hybrid vlan 100 115 tagged
            SW01[ interface GigabitEthernet1/1/1 ]> port hybrid vlan 130 untagged
            SW01[ interface GigabitEthernet1/1/1 ]> port hybrid pvid vlan 130


        >> Configurar um ‘range’ de portas de uma vez <<
            SW01> interface range GigabitEthernet1/1/11 to GigabitEthernet 1/1/22 
            SW01[ interface GigabitEthernet1/1/1 - 22]> port linktype access 
            SW01[ interface GigabitEthernet1/1/1 - 22]> port access vlan 100 
            SW01[ interface GigabitEthernet1/1/1 - 22]> poe enable


--------------------------------------------------SSH---------------------------------------------------------
        >> Habilitar o ssh server
            ssh server enable

        >> Gerar o public key
            public key local create rsa

--------------------------------------------------SNMP---------------------------------------------------------

        >> Configuração SNMP << 
            snmp-agent
            snmp-agent local-engineid 800063A203D07E28BD87F1
            snmp-agent community read public
            snmp-agent community write private
            snmp-agent community write f1m2r3p4
            snmp-agent sys-info contact STI-FMRP
            snmp-agent sys-info location SWITCH_CAMERAS_CORE_STI
            snmp-agent sys-info version all

-----------------------------------------------INFORMAÇÕES do SWITCH---------------------------------------

        >> Pegar o tipo Switch
            display device manuinfo

-------------------------------------------LINK-AGGREGATION-------------------------------------------------
        >> system-view
            link-aggregation load-sharing mode destination-mac source-mac
        
        >> Crie uma interface virtual de agregação de link
            interface bridge-aggregation 11
            link-aggregation mode dynamic

        >> Colocar as interfaces participantes da agregação em modo access
            interface gigabitethernet 0/0/3 
            port link-type access
            port link-aggregation group 11

            interface gigabitethernet 0/0/4 
            port link-type access
            port link-aggregation group 11

        >> Adicionar as vlan´s
            interface bridge-aggregation 11
            port link-type trunk
            undo port trunk permit vlan 1
            port trunk permit vlan 100 115 130 152 to 153 402 410 415 420 430 440
            link-aggregation load-sharing mode destination-mac source-mac

              
