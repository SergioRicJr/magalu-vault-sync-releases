# Magalu Vault Sync

Sincronize notas e anexos do Obsidian com o Object Storage da Magalu Cloud,
usando criptografia no aparelho e histórico de revisões. Este é um **plugin
independente da comunidade**, sem afiliação, certificação ou aprovação da
Magalu Cloud ou do Obsidian.

Versão desta publicação: **0.2.0**. Obsidian mínimo: **1.13.0**.
O programa usa APIs portáteis para Windows e Android. A validação em Android
físico ainda precisa ser concluída; teste primeiro com um cofre fictício.

A preparação de 0.2.0 passou por 173 testes automatizados e pelos
autotestes local e S3 no Obsidian Windows 1.13.7. A integração real conferiu
gravação, leitura limitada com HEAD e HTTP 206, criptografia, integridade e
reenvio. Isso não equivale a aprovação no catálogo ou auditoria independente.

## Requisitos e custos

- Obsidian compatível, plugins da comunidade habilitados e espaço livre no
  aparelho para o cofre e as cópias de recuperação.
- Uma conta Magalu Cloud, um bucket **privado** e credenciais S3 de leitura e
  gravação. O plugin não cria conta, bucket ou credencial.
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
3. Baixe os arquivos da [release 0.2.0](https://github.com/SergioRicJr/magalu-vault-sync-releases/releases/tag/0.2.0)
   e confira `SHA256SUMS.txt`. Os três arquivos instaláveis são `main.js`,
   `manifest.json` e `styles.css`. O ZIP automático de código do GitHub não
   deve ser confundido com um pacote de instalação.
4. Com o Obsidian fechado, copie os três arquivos para:

   ```text
   <SEU_COFRE>/.obsidian/plugins/magalu-vault-sync/
   ```

5. Abra o Obsidian, permita plugins da comunidade e habilite **Magalu Vault
   Sync**. Abra a página de configurações do plugin.

Se a versão já estiver disponível no catálogo do aplicativo, você também
poderá instalá-la por **Configurações → Plugins da comunidade → Explorar**.
Publicar arquivos no GitHub, por si só, não significa que o cadastro no
catálogo tenha sido concluído.

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
2. Instale os três arquivos da mesma versão do plugin.
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

1. Crie no Obsidian um cofre vazio no armazenamento do celular. Para
   instalação manual, escolha uma pasta que seu gerenciador de arquivos ou
   a conexão USB consiga acessar.
2. Abra o cofre uma vez e feche o aplicativo. Habilite a exibição de arquivos
   ocultos no gerenciador de arquivos, se necessário.
3. Copie somente `main.js`, `manifest.json` e `styles.css` para
   `<SEU_COFRE>/.obsidian/plugins/magalu-vault-sync/`. Não coloque uma pasta
   extra entre o nome do plugin e esses arquivos.
4. Reabra o Obsidian e habilite o plugin em **Plugins da comunidade**.
5. Use **Abrir assistente → Conectar ao cofre existente** e escolha a
   mesma região, bucket e prefixo. Cadastre o par S3 do celular e
   importe a chave de recuperação original, usando os formulários separados.
6. Execute os testes do plugin e a primeira sincronização manual. Mantenha
   o Obsidian aberto até o término e confira uma nota e um anexo baixados.
7. Crie uma nota fictícia no celular, sincronize e confira no computador
   depois de sincronizá-lo. Edite-a no computador e repita o caminho
   inverso. Esse teste precisa funcionar antes de usar notas importantes.

Não conte com sincronização enquanto o aplicativo estiver fechado,
suspenso ou limitado pelo Android. Ao trocar de aparelho, sincronize antes
de editar e novamente ao terminar. A execução em segundo plano não é
garantida nem oferecida como serviço pelo plugin.

## Conflitos, exclusões e recuperação

Cada edição gera uma revisão com identificador próprio e referências às
versões anteriores. Leituras e listagens do provedor podem atrasar; uma
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
restaurar, suspenda a sincronização nos demais aparelhos e confira o
conteúdo restaurado antes de retomar o envio. Não apague objetos remotos
manualmente para liberar espaço: revisões posteriores podem depender deles.

Nomes que diferem apenas por maiúsculas/minúsculas ou normalização Unicode
podem ser recusados para evitar colisões entre Windows e Android. Essa
restrição considera o histórico: renomear por um nome intermediário não
remove um nome antigo do histórico.

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
S3 são assinadas. A chave de recuperação não deve ser enviada ao provedor.
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

Faça backup, espere a sincronização terminar, desative o plugin e substitua
somente os três arquivos de programa pela release escolhida. Preserve o
estado local e os segredos. Reative e execute uma sincronização manual antes
de continuar. Não instale versões diretamente de uma branch em movimento.

A versão 0.2.0 migra automaticamente o cache local antigo para a versão 2
antes de iniciar operações de sincronização. O formato de revisões e
envelopes criptografados na nuvem continua na versão 1. **Não faça downgrade
para 0.1.x depois de usar 0.2.0:** as versões antigas não aceitam o cache
local novo. Preserve um backup anterior à atualização e não apague o estado
para tentar forçar o downgrade.

`SHA256SUMS.txt` cobre os arquivos publicados, exceto ele próprio.
`RELEASE-MANIFEST.json` registra a versão e hashes do conjunto de arquivos
distribuídos. Um checksum ajuda a identificar o arquivo baixado, mas não é
uma auditoria do programa nem prova de aprovação por terceiros.

## Licença e fonte

Os arquivos JavaScript e a documentação distribuídos neste repositório
público estão sob **MIT**, copyright SergioRicJr; veja [LICENSE](LICENSE).
O TypeScript de desenvolvimento permanece em um repositório privado. Isso
não limita os direitos MIT de usar, modificar e redistribuir os arquivos
publicados. O JavaScript distribuído pode ser inspecionado normalmente.

O catálogo Obsidian oferece verificação com acesso ao fonte privado por sua
GitHub App. Essa opção não representa aprovação já concedida a esta release
nem auditoria independente. A presença do plugin no catálogo deve ser
conferida no próprio aplicativo. A biblioteca incorporada `aws4fetch` mantém
sua licença MIT e atribuição originais em [NOTICE](NOTICE).
