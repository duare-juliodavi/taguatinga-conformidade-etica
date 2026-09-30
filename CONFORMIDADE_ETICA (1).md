# Conformidade Ética e LGPD – Sistema Taguá

Projeto de Extensão – Proposta de Projetos para a Administração de Taguatinga (UCB, 2026).
Módulos: **Taguá Soluciona**, **Taguá Empreende** e **Evento Fácil**.

## 1. Objetivo

Este documento registra como o sistema Taguá trata dados pessoais e imagens dos cidadãos, em conformidade com a Lei Geral de Proteção de Dados (LGPD, Lei 13.709/2018). Ele reúne o diagnóstico dos dados, a ficha de minimização do DER, as regras de acesso (RBAC), o termo de consentimento e o protocolo de imagem.

## 2. Mapeamento de dados

### 2.1 Dados comuns (3)

| Dado | Onde aparece no DER | Finalidade | Base legal |
|---|---|---|---|
| Nome | `USUARIO.nome` | Identificar o solicitante e gerar o protocolo | Execução de políticas públicas (LGPD, art. 7º, III, e art. 23) |
| CPF ou e-mail | `USUARIO.cpf_email` | Autenticar (Gov.br) e notificar sobre o andamento | Execução de políticas públicas |
| Endereço ou local da demanda | `OCORRENCIA`, `EVENTO.endereco`, `CONSULTA_VIABILIDADE.cep` | Localizar o problema, o evento ou o empreendimento | Execução de políticas públicas |

### 2.2 Dados de atenção reforçada (2)

| Dado | Onde aparece no DER | Risco | Tratamento |
|---|---|---|---|
| Imagem com rosto | `ANEXO.arquivo` | A foto pode captar pessoas, inclusive crianças. Se a imagem fosse usada para identificar alguém, seria dado biométrico, que a LGPD classifica como sensível (art. 5º, II) | Desfoque automático de rostos e placas; a foto original é descartada |
| Geolocalização precisa | `OCORRENCIA.latitude`, `OCORRENCIA.longitude` | Pode revelar a residência ou a rotina do cidadão | Guardar só a coordenada da ocorrência, nunca a do aparelho; remover o GPS embutido na foto |

> Nota: o sistema não coleta de propósito os dados que o art. 5º, II, da LGPD lista como sensíveis (origem racial, saúde, religião, biometria, entre outros). Os dois itens acima são os mais próximos disso e por isso recebem proteção reforçada.

## 3. Ficha de minimização

Campos eliminados, movidos ou reduzidos no DER para coletar apenas o necessário.

| Campo | Decisão | Motivo |
|---|---|---|
| `USUARIO.perfil` | Eliminado | Substituído por `PERFIL` e `USUARIO_PERFIL` (um usuário pode ter mais de um perfil) |
| `LICENCA.status` | Eliminado | A licença só existe após a aprovação; o status já está no evento e nas movimentações |
| `EVENTO.espaco_id` | Eliminado | O uso de espaço da Administração passa pela `RESERVA` |
| `SOLICITACAO.latitude`, `longitude`, `prioridade`, `categoria_id` | Movidos para `OCORRENCIA` | Só fazem sentido em chamados de zeladoria; evita campos vazios nos outros módulos |
| `EMPRESA.razao_social` | Eliminado (proposta) | Pode ser obtida pelo CNPJ em base pública, sem armazenar de novo |
| `NOTIFICACAO.mensagem` | Substituído por tipo de aviso (proposta) | O texto vem de modelos prontos, sem guardar mensagens com dados pessoais |
| `USUARIO.cpf_email` | Reduzido (proposta) | Guardar um único identificador (Gov.br) e exibir o CPF sempre mascarado |
| Foto original | Não armazenada | Só a versão com desfoque é guardada |
| Metadados EXIF da foto (modelo do celular, data, GPS) | Removidos | A localização vem do endereço ou do mapa do sistema |

## 4. Regras de acesso (RBAC)

### 4.1 Perfis

| Grupo | Perfil | Pode | Restrição |
|---|---|---|---|
| Externo (Gov.br) | Cidadão | Abrir solicitações com foto e localização, acompanhar pelo protocolo, anexar documentos, solicitar reserva, receber notificações | Só vê as próprias solicitações |
| Externo (Gov.br) | Empreendedor | Cadastrar empresa, pedir consulta de viabilidade, acompanhar status e resultado | Só vê as consultas das próprias empresas |
| Externo (Gov.br) | Organizador de evento | Solicitar licença eventual, seguir o checklist, enviar documentos, baixar a licença | Só vê os próprios eventos |
| Interno | Servidor operacional | Ver a fila do seu setor, validar a classificação, criar e executar ordens de serviço com fotos antes e depois | Não altera regras de prioridade; não vê outros setores |
| Interno | Analista | Analisar viabilidades e licenças com checklist; aprovar, reprovar ou pedir complementação com justificativa; registrar encaminhamentos; emitir licença | Só a fila do seu módulo |
| Gestão | Gestor regional | Consultar painéis, mapas e relatórios; redistribuir demandas; ver alertas de atraso | Não edita processos |
| Gestão | Administrador | Configurar categorias, prazos (SLA), prioridades, espaços e bloqueios; gerenciar usuários e perfis; consultar a auditoria | É o único que altera as regras do sistema |

### 4.2 Regras gerais

- Acesso restrito ao próprio dado (perfis externos) ou ao próprio setor (perfis internos).
- Toda decisão de servidor e toda ação administrativa gera registro em `MOVIMENTACAO` ou na auditoria.
- Um usuário pode acumular mais de um perfil externo (por exemplo, empreendedor e organizador de evento).
- Princípio do menor privilégio: cada perfil recebe só as permissões de que precisa.

## 5. Termo de Consentimento (TCLE)

**Termo de Consentimento – Taguá Solicita**

**O que é este sistema?**
É um site e aplicativo da Administração de Taguatinga. Por ele você pede serviços, como consertar uma luz queimada, e acompanha o andamento do seu pedido.

**Quais dados a gente pede?**
Seu nome, CPF ou e-mail, o endereço ou local do problema e a descrição do que está acontecendo. Você também pode enviar fotos.

**Por que a gente pede esses dados?**
Só para atender o seu pedido e avisar você sobre ele. Também usamos números gerais, sem mostrar quem você é, para a Administração saber onde estão os problemas.

**E as minhas fotos?**
- Tire foto do problema, não de pessoas.
- Se aparecer rosto ou placa de carro, o sistema borra automaticamente antes de guardar a imagem.
- Os servidores só veem a foto já borrada.

**Quem vê as minhas informações?**
Só os servidores responsáveis pelo seu pedido. Não vendemos nem repassamos seus dados a empresas.

**Por quanto tempo guardamos?**
Pelo tempo necessário para resolver o pedido e cumprir as regras da Administração: **[prazo a definir com a Administração]**.

**Seus direitos**
Você pode perguntar quais dados temos sobre você, pedir correção e pedir a exclusão do que não for obrigatório guardar. Pode também desistir de usar o sistema quando quiser, e isso não atrapalha outros atendimentos seus na Administração.

## 6. Protocolo de imagem

### 6.1 Como fotografar (orientação ao cidadão)

1. Tire duas fotos: uma de longe, mostrando onde fica o problema, e uma de perto, mostrando o defeito.
2. Segure o celular na horizontal e fotografe de dia, se possível. À noite, evite flash em postes e placas, que estouram a imagem.
3. Espere pessoas e carros saírem do quadro. Não fotografe crianças, o interior de casas nem placas de veículos.
4. Limite de **[ex.: 3]** fotos por pedido e **[ex.: 5 MB]** por foto.

### 6.2 Como borrar rostos (processo automático no servidor)

1. Ao receber a foto, o sistema detecta rostos e placas antes de guardar ou exibir qualquer coisa.
2. Em cada área detectada, aplica desfoque gaussiano forte, com o tamanho do desfoque proporcional ao tamanho do rosto (por exemplo, um terço da largura da caixa). Desfoque fraco ou mosaico com blocos pequenos pode ser desfeito.
3. A caixa de detecção é ampliada em 20% a 30% para cobrir testa, orelhas e cabelo.
4. A detecção é configurada para errar por excesso: na dúvida, borra. Isso cobre rostos de perfil, pequenos ou parcialmente cobertos.
5. Só a versão borrada é armazenada e exibida. A original é descartada logo após o processamento (minimização de dados, LGPD).
6. Os metadados EXIF (modelo do celular, data e GPS) são removidos. A localização vem só do campo de endereço ou do mapa do sistema.
7. As fotos de antes e depois feitas pelos servidores seguem o mesmo protocolo.
8. O cidadão pode marcar manualmente uma área para borrar antes de enviar, como reforço da detecção automática.

Sugestão de implementação: OpenCV ou MediaPipe para detectar rostos e OpenCV ou Pillow para aplicar o desfoque, no backend.

## 7. Direitos do titular e retenção

- O cidadão pode pedir confirmação de tratamento, acesso, correção e exclusão dos seus dados (LGPD, art. 18).
- Prazo de guarda: **[a definir com a Administração]**.
- Encarregado de proteção de dados: **[nome e contato]**.

## 8. Pendências

- Validar com a Administração e com o orientador a base legal e o prazo de guarda.
- Confirmar se o teste do protótipo com voluntários exige parecer de comitê de ética da UCB.
- Atualizar o DER com as mudanças marcadas como "proposta" na seção 3.

**Autores:** André Marcos de Souza Batista, Isadora Maia Pascoal Gomes, Jonathas Figueredo Silva, Julio Davi Duarte de Sousa, Samuel Telles de Vasconcellos Resende.
**Orientador:** Marco Antonio de Abreu Machado.
