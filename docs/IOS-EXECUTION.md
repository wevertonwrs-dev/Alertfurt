# AlertFurt iOS — execução pelo Windows e teste no iPhone

Status: especificação e configuração inicial. Aplicativo ainda não implementado, compilado ou testado.
Rota aprovada: GitHub → Codemagic (macOS na nuvem) → App Store Connect → TestFlight.
O repositório é público. Certificados, chaves .p8, senhas e tokens ficam somente nos cofres dos serviços.

## Objetivo e tecnologia
Implementar versão nativa iOS com Swift/SwiftUI, Core Bluetooth, armazenamento local e Keychain.
Priorizar iOS 26+ para avaliar AlarmKit. Confirmar versão do aparelho antes de fixar deployment target.
O usuário acompanha por Windows/iPhone; o ambiente macOS necessário à compilação é remoto.
Não converter o produto em site: entregar aplicativo assinado instalável pelo TestFlight.

## Fluxos completos
1. Abertura: restaurar estado real; nunca exibir protegido apenas por valor salvo.
2. Boas-vindas: explicar Guardião e iniciar configuração ou entrar.
3. Conta: cadastro, confirmação, login, recuperação, perfil, logout e exclusão. Manter entrada local funcional independente da nuvem; autenticação remota não configurada deve ser identificada.
4. Preparação: Bluetooth, notificações, autorização AlarmKit quando disponível, PIN/biometria e teste de áudio.
5. Dispositivos: buscar periféricos com serviço suportado, conectar, conferir protocolo, associar, renomear, reconectar e remover. Um ativo por vez.
6. Teste inicial: prévia sonora e afastamento físico guiado. Simulação claramente separada.
7. Proteção: estado, Guardião, impedimentos, ativar/desativar e último evento.
8. Perda: reconexão, confirmação de ausência, alarme, desarme e resumo do incidente.
9. Histórico: eventos reais, filtros, detalhe, exportação e exclusão confirmada.
10. Ajustes: tolerância, som, vibração quando suportada, privacidade, PIN/biometria e diagnóstico.
11. Ajuda, termos, privacidade e sobre.
12. Saída: ir à tela inicial mantém operação permitida; logout/remover Guardião exige aviso e desarme autenticado. Tratar interrupções e reinício explicitamente.
Cada tela tem loading, vazio, sucesso, erro e retorno. Não incluir botão sem função.

## Núcleo e diferenças essenciais do iOS
Estados: não configurado, pronto, armando, protegido, reconectando, alerta, interrompido.
Apenas conexão/identidade verificadas podem permitir armar.
Usar CBCentralManager, CBPeripheral e serviço GATT conhecido; configurar bluetooth-central e descrição de uso. Implementar preservação/restauração de estado.
Associação não garante acompanhamento permanente. Medir latência real entre perda física, callback e alarme.
Não transportar foreground services, WorkManager ou permissões Android para iOS.
Não implementar tolerância crítica apenas com Timer: suspensão do processo exige estratégia de prazo persistido, reconciliação nos callbacks e mecanismos permitidos pelo sistema. Provar estratégia em aparelho.
Avaliar AlarmKit para agendamento após perda detectada; autorização e agendamento real devem ser testados. Uma notificação comum não equivale a alarme que ignora silencioso.
Controles de parada oferecidos pelo iOS não podem ser substituídos por promessa de PIN obrigatório para todo silenciamento. Distinguir alarme silenciado pelo sistema de proteção desarmada no app.
PIN/biometria protegem comandos internos; reconexão não encerra ocorrência já registrada.
Registrar encerramento/restauração quando observáveis. Sem garantia de funcionamento após force quit.
Nada de áudio silencioso em loop para obter execução permanente.
Sem RSSI convertido em distância exata ou garantia de dez metros.
Internet, login e sincronização não são dependências da proteção local.
AirTag, smartwatch e fones não são presumidos compatíveis.

## Hardware
Guardião de referência nRF52840 DK, protocolo BLE documentado com UUIDs e versão.
Não presumir buzzer/bateria. Alarme no acessório requer componentes e firmware.
Implementar vínculo seguro e identidade; nome anunciado não autentica periférico.
Sem dispositivo físico, continuar implementação e testes de lógica, sem declarar BLE validado.

## Entrega e execução por agente
Criar ios/AlertFurt.xcodeproj, scheme compartilhado AlertFurt e suite AlertFurtTests.
Implementar funções, executar build/testes, corrigir até aprovação e entregar relatório.
Manter documentos e configuração no mesmo repositório.
Codemagic deve primeiro executar testes no simulador e depois gerar IPA assinado para TestFlight. Simulador não valida BLE físico.
Dados de conta/backend ficam em configuração segura; nunca simular autenticação real.
Entregar histórico de build, artefatos e checklist físico: tela bloqueada, afastamento, reconexão, silencioso/Foco, offline, Bluetooth desligado e retorno após interrupção.
Somente declarar pronto para instalar quando houver build processado pela Apple e disponível para o testador.
Não publicar na App Store nesta etapa.

## Configuração externa necessária
- Conta Codemagic conectada ao repositório e à branch ios-bootstrap.
- Apple Developer Program ativo.
- Bundle ID e registro do app no App Store Connect; sugestão ainda não registrada: br.com.alertfurt.ios.
- Integração App Store Connect no Codemagic chamada alertfurt-apple; chave inserida diretamente no serviço, nunca no chat/repositório.
- Certificado Apple Distribution e perfil App Store correspondentes.
- Testador interno e app TestFlight no iPhone.
O arquivo codemagic.yaml é modelo inicial e depende do projeto iOS, testes e credenciais. Não houve build.
