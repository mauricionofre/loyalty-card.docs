# Requisitos Não-Funcionais do Processo de Onboarding (MVP)

## 1. Performance
- **RNF001:** O tempo de resposta da submissão do formulário deve ser inferior a 3 segundos em 95% dos casos
- **RNF002:** O envio de emails deve ocorrer em até 2 minutos após solicitação
- **RNF003:** O sistema deve ser capaz de processar até 100 registros simultâneos de onboarding

## 2. Escalabilidade
- **RNF004:** O design deve permitir escalabilidade futura, mesmo que não implementada no MVP
- **RNF005:** O sistema deve suportar um crescimento de 50% de usuários sem necessidade de redesenho

## 3. Disponibilidade
- **RNF006:** O sistema de onboarding deve manter 99% de disponibilidade durante horário comercial
- **RNF007:** Manutenções programadas devem ser agendadas fora do horário comercial

## 4. Segurança
- **RNF008:** Dados sensíveis devem ser criptografados em trânsito (HTTPS) e senhas em repouso
- **RNF009:** Tokens de validação devem expirar em 7 dias conforme ADR-001
- **RNF010:** Implementação básica de proteção contra ataques comuns (injeção SQL, XSS)

## 5. Auditabilidade
- **RNF011:** Registro básico de operações críticas (criação de tenant, validação de email)
- **RNF012:** Logs de erros detalhados para diagnóstico de problemas

## 6. Usabilidade
- **RNF013:** O formulário de onboarding deve funcionar nos navegadores principais (Chrome, Firefox, Edge, Safari)
- **RNF014:** O processo deve ser responsivo para desktop (adaptação para mobile em versão futura)
- **RNF015:** Mensagens de erro devem ser claras e orientar o usuário

## 7. Recuperabilidade
- **RNF016:** Backups diários do banco de dados
- **RNF017:** Em caso de falha no envio de email, implementar retry simples (3 tentativas)

## 8. Internacionalização
- **RNF018:** MVP com suporte apenas para PT-BR
- **RNF019:** Estrutura de código que permitirá internacionalização futura

## 9. Evolução Pós-MVP
Os seguintes requisitos não-funcionais estão planejados para implementação após o MVP:
- Performance otimizada (tempo de resposta < 1s em 99% dos casos)
- Escalabilidade horizontal automática
- Disponibilidade estendida (99.9%)
- Segurança avançada (MFA, análise de ameaças)
- Auditoria completa e rastreabilidade
- Suporte completo para dispositivos móveis
- Recuperação automatizada de falhas
- Suporte para múltiplos idiomas