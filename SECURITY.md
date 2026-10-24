# Segurança

Este projeto distribui o plugin Magalu Vault Sync. Consulte a versão exata
instalada e as limitações em [README.md](README.md). Os arquivos públicos são
MIT; o desenvolvimento TypeScript fica em um repositório privado. Não há
garantia de ausência de falhas nem auditoria independente anunciada.

## Comunicar uma vulnerabilidade

Não publique notas, anexos, logs completos, configurações do cofre, nomes de
bucket, credenciais S3, tokens, exportações de SecretStorage ou a chave de
recuperação em issues, comentários ou screenshots.

Se a opção de relato privado estiver habilitada, use
[Report a vulnerability](https://github.com/SergioRicJr/magalu-vault-sync-releases/security/advisories/new).
Se ela não estiver disponível, abra uma issue solicitando um canal privado de
segurança, sem incluir detalhes exploráveis ou dados pessoais. O mantenedor
indicará como continuar. Não envie credenciais para demonstrar um problema.

Informe apenas a versão do plugin e do Obsidian, plataforma, descrição do
impacto e uma reprodução com um cofre fictício. Um arquivo de exemplo deve
conter somente conteúdo criado para o teste. Antes de anexar qualquer coisa,
revise o nome do arquivo, propriedades, texto, URLs e imagens.

## Se houver suspeita de exposição

Suspenda o plugin nos aparelhos e preserve uma cópia local do cofre. Revogue
no provedor a credencial que possa ter sido exposta e crie outra antes de
retomar o uso. Alterar a credencial S3 não altera a chave de recuperação nem
recifra os objetos já gravados. Uma chave de recuperação exposta exige um
procedimento de migração de dados; não gere outra sobre o mesmo prefixo como
tentativa de corrigir o problema.

O SecretStorage do Obsidian é compartilhado com o ambiente dos plugins e não
fornece isolamento contra outros plugins ou contra um processo local
malicioso. Mantenha backups independentes, proteja os dispositivos e habilite
somente plugins em que confia.

## Versões

Esta publicação descreve a versão 0.2.1. Use uma release fixa, confira
os hashes publicados e leia as mudanças antes de substituir os arquivos. O
plugin não instala atualizações nem dependências automaticamente. A presença
no catálogo ou um resultado de scanner não equivale a endosso do provedor ou
a auditoria de segurança independente.
