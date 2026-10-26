# Magalu Vault Sync

Sincronize notas e anexos do Obsidian com o Object Storage da Magalu Cloud,
usando criptografia no aparelho e histórico de revisões. Este é um **plugin
independente da comunidade**, sem afiliação com a Magalu Cloud ou o Obsidian.

**Disponível no [catálogo da comunidade](https://community.obsidian.md/plugins/magalu-vault-sync).**
No Obsidian, use **Configurações → Plugins da comunidade → Explorar → Magalu
Vault Sync → Instalar → Habilitar**. Confira o autor **SergioRicJr**.

Versão desta publicação: **0.2.2**. Obsidian mínimo: **1.13.0**.
O programa usa APIs portáteis para Windows e Android. A validação física
específica da versão final 0.2.2 no Android ainda não está registrada;
teste primeiro com um cofre fictício.

O projeto passou por mais de 200 testes automatizados, incluindo uma
regressão de Base64 e AES-GCM com anexo de 20 MiB. A versão 0.2.1 teve
sincronização confirmada em Windows e Android, incluindo recebimento no
computador de uma nota fictícia criada no celular. Consulte as notas de
cada release para os resultados da versão exata. A 0.2.2 foi publicada no
catálogo após a revisão automática, sem erros bloqueantes. Essa revisão e
os testes não equivalem a auditoria independente ou certificação de segurança.

## Encontre a orientação certa

- [Instalar pelo catálogo ou manualmente](#instalar-no-computador)
- [Usar um cofre que já tem notas](#cofres-locais-já-existentes)
- [Configurar o primeiro aparelho](#configurar-pela-primeira-vez)
- [Conectar outro computador](#outro-computador-ou-aparelho-novo)
- [Usar no Android](#android)
- [Testar a ida e volta das notas](#teste-entre-computador-e-celular)
- [Entender a arquitetura e as chaves](#arquitetura-e-conexão)
- [Resolver conflitos e recuperar arquivos](#conflitos-exclusões-e-recuperação)
- [Diagnosticar problemas](#resolver-problemas)
- [Atualizar preservando a configuração](#atualizar-e-conferir-a-publicação)

## Arquitetura e conexão

O plugin roda dentro do Obsidian e faz requisições de saída diretamente ao
Object Storage da Magalu. Não há backend próprio, servidor intermediário,
servidor local ouvindo portas ou serviço independente em segundo plano.

```text
Obsidian + plugin no computador
              ↕ HTTPS / API S3
Bucket privado na Magalu: revisões criptografadas
              ↕ HTTPS / API S3
Obsidian + plugin no Android
```

Os aparelhos não precisam estar ligados ao mesmo tempo. Um envia as revisões
e o outro as consulta depois. GitHub e catálogo distribuem o programa; não
participam do transporte das notas.

O transporte usa a API HTTP nativa do Obsidian e assina as requisições com
AWS Signature Version 4. Lista e lê objetos, confere tamanhos e envia novas
revisões. A conta precisa de permissões de **listar, ler e gravar** no destino.
O plugin não precisa excluir objetos remotamente. O autoteste usa um
subprefixo separado dentro do prefixo configurado, que também precisa dessas
permissões. São aceitos somente os endpoints regionais oficiais oferecidos
nas configurações, com HTTPS.

| Valor | Função |
| --- | --- |
| Access Key ID | Identifica a credencial de acesso ao S3. |
| Secret Access Key | Assina as requisições S3; não é enviada como segredo em texto nos cabeçalhos. |
| Chave de recuperação | Criptografa e descriptografa as revisões no aparelho; não é enviada pela sincronização à Magalu. |

A chave de recuperação é uma chave AES aleatória de 32 bytes representada
por 43 caracteres. Não é a senha da conta Magalu. Todos os aparelhos do
mesmo cofre remoto precisam da chave original. **Sem ela, o plugin não
recupera o conteúdo criptografado remoto.** Não há recuperação de senha,
rotação ou migração automática da chave no mesmo destino. Guarde uma cópia
independente em um gerenciador de senhas; não a envie ao suporte.

## Como a sincronização funciona

As notas continuam sendo arquivos locais comuns. Escreva nas pastas normais
do cofre; não existe uma pasta especial obrigatória para enviar notas.
Arquivos e pastas com ponto inicial, a configuração do Obsidian e a
recuperação local ficam fora da sincronização.

1. O plugin compara os arquivos locais com o último estado conhecido e
   registra a intenção em uma fila persistente antes de tentar enviar.
2. Cada alteração detectada numa rodada vira uma revisão com UUID e
   referências às revisões anteriores. Eventos próximos são agrupados;
   não há uma revisão para cada tecla ou salvamento. O arquivo alterado é
   enviado por inteiro; não são deltas de texto.
3. Conteúdo, caminho e metadados são criptografados no aparelho antes do S3.
   O bucket recebe objetos com identificadores, sem os nomes das suas notas.
4. O outro aparelho lista as revisões, autentica e descriptografa as que
   precisa conhecer ou aplicar, e preserva alterações concorrentes.
5. Antes de substituir ou remover um arquivo local, registra a operação e
   mantém uma cópia de recuperação. Exclusões remotas são revisões do
   histórico; o plugin não apaga objetos definitivamente.

Na versão 0.2.2, uma rodada comprovadamente inalterada pode usar os IDs já
autenticados em cache, sem baixar novamente os conteúdos nem regravar notas.
Ainda há leitura local, listagem remota e persistência do horário da rodada.
Isso pressupõe IDs de revisão imutáveis e não é uma auditoria contínua de
todo o ciphertext armazenado. Novas revisões e conteúdos baixados continuam
exigindo validação e autenticação antes da aplicação.

A fila e parte do estado local podem conter conteúdo em Base64, que é uma
representação legível, não criptografia. Não copie o estado entre aparelhos
nem o publique. A credencial S3 autoriza o armazenamento; a chave de
recuperação abre o conteúdo criptografado. Guarde essa chave fora do cofre.

Um aparelho novo precisa inspecionar revisões desconhecidas para reconstruir
o histórico; isso pode incluir versões antigas de arquivos. O histórico
cresce, não há expurgo automático e a transferência inicial pode ser maior
que o tamanho das notas atuais. Conteúdos temporários de descoberta podem
ser descartados e relidos para manter o orçamento de trabalho.

## Requisitos e custos

- Obsidian compatível, plugins da comunidade habilitados e espaço livre no
  aparelho para o cofre e as cópias de recuperação.
- Uma conta Magalu Cloud, um bucket **privado** e credenciais S3 para listar,
  ler e gravar objetos. O plugin não cria conta, bucket ou credencial.
- Internet para enviar e receber alterações. As notas locais continuam
  disponíveis offline.
- Um gerenciador de senhas para guardar uma cópia independente da chave de
  recuperação.

O plugin não cobra pelo uso. A Magalu pode cobrar armazenamento,
requisições e transferência conforme sua contratação, incluindo histórico e
objetos de teste. A compatibilidade é limitada aos endpoints regionais
oficiais oferecidos nas configurações. Não é um cliente para qualquer
serviço que aceite S3. Consulte os [endpoints](https://docs.magalu.cloud/docs/storage/object-storage/additional-explanations/regions)
e a cobrança na documentação do provedor.

## Instalar no computador

1. Faça backup independente do cofre. Comece com um cofre de teste e notas
   fictícias antes de habilitar no seu material de trabalho.
2. Escolha uma pasta local fora de OneDrive, Google Drive, Dropbox ou outro
   sincronizador. Use somente um mecanismo de sincronização por pasta.
3. No cofre escolhido, abra **Configurações → Plugins da comunidade →
   Explorar**. Procure **Magalu Vault Sync**, autor **SergioRicJr**, e use
   **Instalar → Habilitar**. Também é possível abrir a
   [página do catálogo](https://community.obsidian.md/plugins/magalu-vault-sync)
   e usar **Add to Obsidian**, escolhendo o cofre correto no aplicativo.
4. Abra **Configurações → Magalu Vault Sync**. A instalação e configuração
   são feitas por cofre; outro cofre exige habilitar o plugin nele também.

### Alternativa: instalação manual

1. Baixe o pacote `magalu-vault-sync-0.2.2.zip` da
   [release 0.2.2](https://github.com/SergioRicJr/magalu-vault-sync-releases/releases/tag/0.2.2)
   e confira seu arquivo `.zip.sha256`. Ele contém a pasta
   `magalu-vault-sync` com três arquivos de programa, LICENSE e NOTICE.
   Também é possível baixar `main.js`, `manifest.json` e `styles.css`
   individualmente e conferir `SHA256SUMS.txt`. O ZIP automático “Source
   code” do GitHub é diferente do pacote de instalação.
2. Com o Obsidian fechado, extraia a pasta do plugin para
   `<SEU_COFRE>/.obsidian/plugins/`. Os três arquivos de programa devem ficar
   diretamente em:

   ```text
   <SEU_COFRE>/.obsidian/plugins/magalu-vault-sync/
   ```

3. Abra o Obsidian, permita plugins da comunidade e habilite **Magalu Vault
   Sync**. Abra a página de configurações do plugin.

## Cofres locais já existentes

É possível usar um cofre que já tem notas, sem mover os arquivos para uma
pasta especial. Faça um backup independente antes da primeira sincronização.
O nome das opções do assistente se refere ao **destino remoto**, não à
criação de uma nova pasta local.

| Situação | O que escolher |
| --- | --- |
| Estas notas ainda não usam o plugin e terão uma sincronização própria | **Começar novo cofre**, com prefixo remoto exclusivo e chave própria. |
| Quero acessar as mesmas notas que outro aparelho já sincroniza | **Conectar ao cofre existente**, com a mesma região, bucket, prefixo e chave original. |
| Este aparelho já está configurado e sincronizando | Continue usando **Sincronizar agora**; uma atualização não exige refazer o assistente. |

Para receber um cofre remoto em um aparelho novo, começar com uma pasta
local vazia simplifica a conferência. Se os dois lados já tiverem notas,
arquivos diferentes no mesmo caminho podem gerar conflitos. Compare e
resolva essas cópias antes de continuar o trabalho. Cofres independentes
podem compartilhar um bucket, mas precisam de prefixos diferentes.

## Configurar pela primeira vez

1. Crie ou escolha um bucket privado e um prefixo exclusivo para este cofre.
   **Não reutilize o prefixo de um cofre diferente.** O nome de bucket usado
   nos seus aparelhos é uma escolha sua; este repositório não contém uma
   configuração de conta pronta.
2. Nas configurações do plugin, use **Abrir assistente → Começar novo
   cofre**. Na etapa 1 de 4, escolha **Região**, confira o endpoint e
   preencha **Bucket privado** e **Pasta remota (prefixo)**. Salve o destino.
3. Na etapa 2, use **Cadastrar credencial S3** para registrar o Access Key ID e o Secret
   Access Key do mesmo par S3. Eles são fornecidos pela Magalu; não são a
   senha da conta nem a chave de recuperação do plugin.
4. Somente no primeiro aparelho de um cofre remoto **novo**, na etapa 3,
   use **Criar e guardar chave**. Guarde a chave em um gerenciador de senhas e
   confirme que salvou a cópia. Ela será necessária em todos os aparelhos
   para abrir as revisões criptografadas.
5. Execute **Testar proteção local** nas configurações e, na etapa 4 do
   assistente, **Testar conexão e criptografia**.
   Eles usam arquivos ou objetos de teste e não substituem o teste de
   sincronização entre aparelhos. Os objetos de teste podem permanecer no
   bucket, pois o plugin não faz exclusão remota definitiva.
6. Use **Sincronizar agora**. Espere o término e confira os avisos de
   pendências, conflitos e arquivos ignorados. A sincronização automática
   exige a primeira execução manual bem-sucedida e só atua quando habilitada.

Prefira uma credencial por aparelho para permitir revogação independente.
Use as menores permissões disponíveis: acesso somente ao bucket pretendido
quando a sua conta oferecer esse isolamento. Uma chave pessoal com nome
exclusivo pode continuar acessando outros buckets do titular. Criptografar
as notas não reduz os privilégios da credencial. Este protocolo grava
exclusões como revisões novas e não precisa apagar objetos remotos.

O versionamento do bucket é uma proteção adicional, com custos próprios;
não é o mesmo mecanismo que o histórico do plugin. Veja a
[documentação de versionamento](https://docs.magalu.cloud/docs/storage/object-storage/how-to/versioning/activate-check-versioning).

## Outro computador ou aparelho novo

1. Instale uma versão compatível do Obsidian e crie um cofre local **vazio**.
2. Instale e habilite o plugin pelo catálogo nesse cofre. Use uma versão
   compatível em todos os aparelhos; a instalação manual é alternativa.
3. Use **Abrir assistente → Conectar ao cofre existente** e configure a
   mesma região, bucket e prefixo do primeiro aparelho.
4. Cadastre a credencial S3 destinada ao novo aparelho.
5. Na etapa 3, use **Importar chave original** e informe a chave **original**
   guardada fora do cofre. Não gere uma chave nova para o prefixo existente.
6. Sincronize, espere o download e confira notas e anexos antes de editar.

Não copie `data.json`, SecretStorage ou a pasta completa de configurações de
outro aparelho. Cada instalação mantém sua própria identificação e estado.
Se o aparelho novo já tiver notas, faça backup antes: conteúdos diferentes
no mesmo caminho podem virar ramos de conflito.

## Android

1. Crie no Obsidian um cofre local vazio para receber as notas do computador,
   ou siga as orientações para um cofre já existente, mantendo um backup.
2. Nesse cofre, use **Configurações → Plugins da comunidade → Explorar →
   Magalu Vault Sync → Instalar → Habilitar**. O Obsidian precisa atender à
   versão mínima indicada no início deste documento.
3. Abra **Configurações → Magalu Vault Sync → Abrir assistente → Conectar ao
   cofre existente** e escolha a
   mesma região, bucket e prefixo. Cadastre o par S3 do celular e
   importe a chave de recuperação original, usando os formulários separados.
4. Execute os testes do plugin e a primeira sincronização manual. Mantenha
   o Obsidian aberto até o término e confira uma nota e um anexo baixados.
5. Crie uma nota fictícia no celular, sincronize e confira no computador
   depois de sincronizá-lo. Edite-a no computador e repita o caminho
   inverso. Esse teste precisa funcionar antes de usar notas importantes.

Não conte com sincronização enquanto o aplicativo estiver fechado,
suspenso ou limitado pelo Android. Ao trocar de aparelho, sincronize antes
de editar e novamente ao terminar. A execução em segundo plano não é
garantida nem oferecida como serviço pelo plugin.

Na instalação manual no Android, abra o cofre uma vez, feche o aplicativo e
extraia a pasta do ZIP para `<SEU_COFRE>/.obsidian/plugins/`. Habilite arquivos
ocultos no gerenciador se necessário. `main.js`, `manifest.json` e `styles.css`
devem ficar diretamente em `plugins/magalu-vault-sync/`, sem uma pasta extra
entre eles. Não copie o estado nem os segredos de outro aparelho.

## Teste entre computador e celular

1. Em cada aparelho, execute **Testar proteção local** e **Testar conexão e
   criptografia**. Eles verificam componentes; não comprovam a sincronização
   entre os aparelhos.
2. No computador, crie uma nota fictícia `Teste de sincronização` com uma
   frase de exemplo e um anexo pequeno sem informações pessoais.
3. Use **Sincronizar agora** no computador e depois no celular. Confira o
   texto e abra o anexo recebido.
4. Acrescente outra frase no celular, sincronize nele e depois no computador.
   Confira se a alteração voltou ao computador.
5. Depois desse teste, habilite **Sincronização automática** se desejar.
   Após a primeira rodada manual bem-sucedida, ela reage às alterações
   agrupadas, à retomada do aplicativo e a uma verificação a cada dois minutos.

No computador, **Ctrl+P → Magalu Vault Sync: Sincronizar agora** oferece o
mesmo comando do botão nas configurações e do ícone de nuvem. Aguarde a
conclusão antes de fechar o aplicativo ou mudar de aparelho. Não edite a
mesma nota nos dois aparelhos sem sincronizar entre as edições.

## Conflitos, exclusões e recuperação

Cada alteração detectada numa rodada gera uma revisão com identificador
próprio e referências às versões anteriores. Leituras e listagens do provedor podem atrasar; uma
listagem vazia não é interpretada como exclusão. Revisões concorrentes são
preservadas. O relógio do aparelho não escolhe qual texto descartar.

Quando surgir uma cópia com ` (conflict IDENTIFICADOR)` no nome, compare as
versões e reúna o conteúdo desejado na nota do caminho original. Com ela
aberta, use o comando **Usar conteúdo atual para resolver conflito da nota**,
confirme sua escolha e sincronize. Preserve as cópias até verificar o
resultado nos outros aparelhos. Editar uma cópia de conflito a transforma
em outra nota; isso sozinho não resolve os ramos da nota original.

Antes de substituir ou remover um arquivo local, o plugin procura preservar
uma cópia em `.magalu-sync-recovery/`, dentro do cofre. Faça backup antes de
restaurar. Para recuperar uma cópia local:

1. Em **Configurações → Plugins da comunidade**, desative **Magalu Vault
   Sync** em todos os aparelhos. Preserve uma cópia da nota atual, se existir.
2. No gerenciador de arquivos, habilite a exibição de pastas ocultas e abra
   `.magalu-sync-recovery/` dentro da pasta do cofre. As cópias são organizadas
   por data, identificador e caminho original.
3. Copie a versão escolhida para um local separado e confira o conteúdo.
   Restaure-a no caminho desejado somente depois de guardar a versão atual.
4. Reative o plugin primeiro em um aparelho e use **Sincronizar agora**.
   Confira o resultado antes de reativar e sincronizar os demais.
   A restauração pode produzir uma nova revisão
   ou exigir resolver um conflito.

Essa pasta fica somente no aparelho que produziu a cópia; não substitui um
backup independente. Não apague objetos remotos
manualmente para liberar espaço: revisões posteriores podem depender deles.

Nomes que diferem apenas por maiúsculas/minúsculas ou normalização Unicode
podem ser recusados para evitar colisões entre Windows e Android. Essa
restrição considera o histórico: renomear por um nome intermediário não
remove um nome antigo do histórico.

## Resolver problemas

| Sintoma | O que conferir |
| --- | --- |
| O plugin não aparece ou não carrega | Versão mínima do Obsidian, plugins da comunidade habilitados e cofre correto. Na instalação manual, confira a pasta sem níveis extras. |
| Falha de rede ou acesso negado | Internet, região, bucket, prefixo e o par correto de credenciais S3; confira as permissões de listar, ler e gravar, incluindo o subprefixo de teste. Não torne o bucket público para tentar corrigir. |
| Erro ao abrir conteúdo criptografado | Importe a chave original do mesmo destino. Não gere outra chave nem apague o estado como tentativa de reparo. |
| Uma alteração ainda não chegou | Aguarde a propagação, sincronize o aparelho que editou e depois o que deve receber. Confira pendências e se o Obsidian está ativo. |
| Nada enviado ou recebido | Pode ser normal se os aparelhos já estiverem atualizados. Confira o texto de uma nota de teste nos dois lados. |
| O automático não executa | Ative a opção, conclua uma primeira sincronização manual e mantenha o aplicativo aberto. |
| Arquivo ignorado ou limite atingido | Leia o aviso e os limites abaixo. Preserve os dados; não apague revisões do bucket nem `data.json` para liberar espaço. |
| Cópia com `conflict` no nome | Compare as versões e use o comando de resolução na nota original, conforme a seção anterior. |

Para pedir ajuda, informe versões, plataforma, aviso curto sem segredos e
passos com dados fictícios. Não publique notas reais, configurações ou logs
completos. Suspeitas de vulnerabilidade seguem [SECURITY.md](SECURITY.md).

## Limites da versão

Os limites são proteções de recursos, não metas de capacidade testada para
qualquer cofre. Quando houver recusa ou aviso, preserve os dados e leia a
mensagem; não apague estado ou revisões para tentar contornar a proteção.

| Recurso | Limite |
| --- | --- |
| Arquivo individual | 20 MiB |
| Arquivos locais considerados | 10.000 |
| Revisões remotas conhecidas/listadas | 10.000 |
| Relações entre revisões | 50.000 |
| Revisões na fila local de envio | 1.000 |
| Conjunto de trabalho de conteúdo não criptografado por sincronização | 64 MiB |
| Revisão serializada | 28 MiB |
| Envelope remoto criptografado | 40 MiB |
| Página de listagem remota | 4 MiB |

Revisões contam alterações, não somente notas; o histórico pode atingir o
limite antes do número de arquivos. Esta versão não expurga o histórico.
Verificações de tamanho de respostas ocorrem depois do recebimento pela API
HTTP nativa, que pode armazenar o corpo antes da validação do plugin; esses
limites não garantem um teto rígido de memória ou transferência de rede.

## Privacidade e modelo de confiança

Conteúdo, caminhos e metadados de cada revisão são criptografados no
aparelho com AES-256-GCM antes do envio. A conexão é HTTPS e as requisições
S3 são assinadas. A chave de recuperação não é enviada ao provedor pela sincronização.
O provedor ainda pode observar número, tamanhos aproximados e horários dos
objetos, além dos dados de conta e autenticação necessários ao serviço.

O plugin utiliza a rede para se comunicar com o Object Storage configurado
e realizar seus testes de conexão. Não inclui telemetria, servidor
intermediário ou atualizador automático. O histórico e os objetos de teste
ocupam armazenamento e geram requisições na sua conta.

Os valores S3 e a chave de recuperação são guardados no **SecretStorage do
Obsidian**; a configuração do plugin contém referências. Esse armazenamento
não isola segredos de outros plugins nem garante um cofre criptográfico do
sistema operacional. Seus arquivos, a recuperação e parte do estado local
continuam legíveis no aparelho. Proteja os dispositivos e seus backups.

Participantes com a chave de recuperação e acesso de escrita podem criar
revisões válidas. O plugin não fornece permissões por pessoa, defesa contra
um aparelho comprometido ou detecção garantida de remoção de todo o
histórico em uma instalação nova. Para binários há verificações e cópias de
recuperação, mas a API portátil não oferece substituição atômica completa
contra gravações simultâneas de outros programas. Evite editar o mesmo
anexo fora do Obsidian durante a sincronização.

Nunca envie notas, credenciais, chave de recuperação, estado, logs completos
ou screenshots do cofre para uma issue. Use dados fictícios e consulte
[SECURITY.md](SECURITY.md) para relatar problemas. Sincronização e histórico
não substituem um backup independente.

## Atualizar e conferir a publicação

Faça backup e espere a sincronização terminar. Em **Configurações → Plugins
da comunidade**, use **Verificar atualizações** e **Atualizar** ao lado do
plugin. Confira a versão instalada, reative-o se necessário e execute uma
sincronização manual. A atualização mantém o mesmo destino e chave; não
exige desinstalar o plugin ou repetir o assistente.

Na alternativa manual, feche o Obsidian e substitua somente `main.js`,
`manifest.json` e `styles.css` pela release escolhida. **Preserve `data.json`,
notas e armazenamento de segredos.** Reabra e confira a primeira rodada.
Não instale versões diretamente de uma branch em movimento.

A versão 0.2.0 migra automaticamente o cache local antigo para a versão 2
antes de iniciar operações de sincronização. O formato de revisões e
envelopes criptografados na nuvem continua na versão 1. **Não faça downgrade
para 0.1.x depois de usar 0.2.0:** as versões antigas não aceitam o cache
local novo. Preserve um backup anterior à atualização e não apague o estado
para tentar forçar o downgrade.

`SHA256SUMS.txt` cobre os arquivos do conjunto que o acompanha, exceto ele
próprio; `RELEASE-MANIFEST.json` registra seus tamanhos e hashes. Para conferir
uma release, use os arquivos e hashes da **mesma tag/release**. A documentação
da branch principal pode receber correções sem alterar os arquivos já
lançados: os manifests da branch descrevem esse snapshot, e os anexos da
release permanecem fixos. Um checksum ajuda a identificar o arquivo baixado,
mas não é uma auditoria do programa nem prova de aprovação por terceiros.

## Licença e fonte

Os arquivos JavaScript e a documentação distribuídos neste repositório
público estão sob **MIT**, copyright SergioRicJr; veja [LICENSE](LICENSE).
O TypeScript de desenvolvimento permanece em um repositório privado. Isso
não limita os direitos MIT de usar, modificar e redistribuir os arquivos
publicados. O JavaScript distribuído pode ser inspecionado normalmente.

O catálogo Obsidian oferece verificação com acesso ao fonte privado por sua
GitHub App. A publicação e a revisão automática da 0.2.2 foram concluídas;
isso não confirma um selo adicional de reprodução do fonte privado nem
auditoria independente. Consulte a
[entrada pública](https://community.obsidian.md/plugins/magalu-vault-sync)
e a versão disponível no aplicativo. A biblioteca incorporada `aws4fetch` mantém
sua licença MIT e atribuição originais em [NOTICE](NOTICE).
