# carweb
CarWeb é um sistema web de compra, venda e aluguel de veículos. Anunciantes e lojas publicam carros com fotos; compradores buscam, favoritam, simulam financiamento e reservam por dia ou por hora. Inclui cadastro em lote por XML, perfis de acesso e segurança. Feito em PHP, MySQL e XAMPP.

# CarWeb

Sistema web para **compra, venda e locação de veículos**. Projeto acadêmico de Ciências da Computação (UNINOVE, 2026), desenvolvido por um grupo de 8 alunos, com liderança de Matheus Costa Machado.

Feito em **PHP 8.0+ puro** (sem framework, padrão MVC), **MySQL/MariaDB** (PDO) e **XAMPP**.

## Funcionalidades

- **Comprar e vender:** anunciantes e lojas publicam veículos com foto de capa e até 8 fotos adicionais (carrossel). Compradores buscam com filtros, favoritam e demonstram interesse. Nenhum pagamento é processado.
- **Alugar:** reserva por **dia** ou por **hora**, com bloqueio de datas em conflito e estimativa de valor.
- **Simulador:** financiamento pela Tabela Price (venda) e parcelas sem juros (aluguel).
- **Cadastro em lote:** importação de até 100 veículos por arquivo **XML**, com modelo para baixar.
- **Tipos de conta:** Comprador, Anunciante, Loja e Administrador geral, cada um com suas permissões. Anunciante e Loja só anunciam: não veem "Alugar" nem "Minhas reservas".
- **Diagnóstico:** a página `/diagnostico` (só em localhost) confere PHP, pastas, banco e tabelas.

## Instalação no XAMPP (Windows)

1. Copie esta pasta para `C:\xampp\htdocs\carweb`.
2. No painel do XAMPP, inicie **Apache** e **MySQL**.
3. No phpMyAdmin (`http://localhost/phpmyadmin`), aba **Importar**, importe `database/schema.sql`.
4. Abra `database/usuario_banco.sql`, **troque a senha** `troque_esta_senha` e execute o script na aba **SQL**. Ele cria o usuário `carweb_app`, que só tem SELECT, INSERT, UPDATE e DELETE no banco `carweb`.
5. Copie `app/config/config.local.example.php` para `app/config/config.local.php` e coloque a mesma senha. Esse arquivo não deve ir para o git.
6. *(Opcional)* Crie contas de demonstração: `C:\xampp\php\php.exe database\seed.php`.
7. Acesse `http://localhost/carweb/`.

Contas de demonstração (senha `Carweb@123`; troque ou apague antes de qualquer uso real): `admin@carweb.local`, `anunciante@carweb.local`, `cliente@carweb.local`.

Sem XAMPP: `php -S 127.0.0.1:8080 -t public public/index.php`.

## Atualizando um banco antigo

Execute `database/migracao_tudo.sql` no phpMyAdmin (banco `carweb` selecionado, aba **SQL**). Ele atualiza qualquer versão anterior e pode ser executado várias vezes. Adiciona o tipo de conta Loja, o aluguel por hora, os favoritos e as fotos do carrossel.

Para apagar os veículos de demonstração, execute `database/limpar_dados_demo.sql`. Ele apaga só os veículos das contas de demonstração.

## Importar veículos por XML

Em **Meus anúncios → Importar XML**, baixe o modelo (`database/modelo_veiculos.xml`), preencha e envie. O arquivo é validado por inteiro com as mesmas regras do cadastro individual. Se houver qualquer erro, **nada é importado** e todos os problemas são listados. Limites: 100 veículos e 1 MB, sem `DOCTYPE`/`ENTITY` (proteção contra XXE).

## Solução de problemas

| Sintoma | O que fazer |
|---|---|
| **Erro 500** | Quando você acessa por `localhost`, a tela mostra o "Detalhe técnico". O motivo também fica em `storage/logs/app.log`. |
| **500 ao anunciar** ou tipos de conta que não separam | Banco desatualizado: execute `database/migracao_tudo.sql`. |
| **Erro 1045** (Access denied para `carweb_app`) | O usuário do banco não existe ou a senha difere: refaça o passo 4 e confira `config.local.php`. |
| **403 Proibido** | Confira se existe `htdocs\carweb\public\index.php` (cuidado com pasta duplicada) e se o `mod_rewrite` do Apache está ativo. |
| **Antes de tudo** | Abra `http://localhost/carweb/public/diagnostico`: cada item em vermelho diz o que corrigir. |

## Segurança

Senhas com `password_hash`, sessão reforçada, token CSRF em todos os formulários, consultas preparadas (PDO), escape de saída, política CSP sem scripts inline, validação de uploads por conteúdo (e não só pela extensão), permissões por perfil, código de segurança para criar administrador e usuário de banco com privilégios mínimos. Em produção, use HTTPS e troque todas as senhas padrão.

## Testes

```
php tests/run.php
```

Executa 28 testes automáticos: cálculo de locação e de financiamento, conflito de datas, validação de CPF, e-mail e veículo, hash de senha, permissões e importação de XML.

## Estrutura

```
public/      única pasta exposta ao navegador (index.php, assets, uploads)
app/core/    rotas, autenticação, CSRF, banco (PDO), regras de negócio, importador de XML, upload
app/models/  acesso a dados
app/controllers/  lógica de cada tela
app/views/   telas e partes reutilizáveis
database/    schema, migrações, modelo de XML, scripts de conta e de limpeza
scripts/     backup do banco (backup.bat, backup.sh)
storage/     logs (fora do alcance do navegador)
tests/       testes automáticos
```

## Backup

`scripts/backup.bat` (Windows) e `scripts/backup.sh` (Linux) geram uma cópia do banco com `mysqldump`.
