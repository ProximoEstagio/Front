# Emails transacionais — Próximo Estágio

Templates HTML de email alinhados ao frontend do Próximo Estágio. Foram usados a paleta do projeto (`#ae0f10`, `#8d0b0b`, `#4b555c`, `#e1e1e1`) e uma pilha de fontes com Montserrat e fallbacks. O CSS é inline e a estrutura usa tabelas para ter compatibilidade maior com clientes de email.

## Templates

| Arquivo | Finalidade | Assunto sugerido |
|---|---|---|
| `conta-criada.html` | Conta criada | Sua conta no Próximo Estágio foi criada |
| `confirmar-email-login.html` | Confirmar email / tentativa de login | Confirme seu email |
| `confirmar-troca-email.html` | Confirmar troca de email | Confirme a troca de email |
| `confirmar-troca-senha.html` | Confirmar solicitação de troca de senha | Confirme a alteração de senha |
| `recuperacao-senha.html` | Recuperação de senha | Recuperação de senha |
| `documento-recebido-professor.html` | Professor notificado sobre novo documento | Novo documento recebido para avaliação |
| `documento-avaliado-aluno.html` | Aluno notificado sobre avaliação | Seu documento foi avaliado |
| `prazo-envio-chegando.html` | Lembrete de prazo ao aluno | Lembrete: prazo de envio se aproximando |
| `estagio-concluido-aluno.html` | Aluno notificado do encerramento bem-sucedido | Seu estágio foi validado com sucesso |
| `estagio-encerrado-secretaria.html` | Secretaria notificada do encerramento de um processo | Processo de estágio encerrado |

## Integração com Spring Boot + Thymeleaf

Copie os arquivos HTML para o classpath do backend, por exemplo em `src/main/resources/templates/emails/`. Adicione/use o starter Thymeleaf e o starter de email do Spring Boot no backend.

Exemplo de renderização:

```java
import org.springframework.stereotype.Service;
import org.thymeleaf.TemplateEngine;
import org.thymeleaf.context.Context;

@Service
public class EmailTemplateRenderer {
    private final TemplateEngine templateEngine;

    public EmailTemplateRenderer(TemplateEngine templateEngine) {
        this.templateEngine = templateEngine;
    }

    public String render(String templateName, java.util.Map<String, Object> variables) {
        Context context = new Context(java.util.Locale.forLanguageTag("pt-BR"));
        context.setVariables(variables);
        return templateEngine.process("emails/" + templateName, context);
    }
}
```

Envie o HTML renderizado como conteúdo HTML do email, por exemplo com `MimeMessageHelper.setText(html, true)`. Configure SMTP e remetente no backend. Não envie senhas ou tokens de sessão permanentes por email.

### Exemplo de uso

```java
var variables = new java.util.HashMap<String, Object>();
variables.put("nome", usuario.getNome());
variables.put("email", usuario.getEmail());
variables.put("urlDefinirSenha", urlDefinirSenha); // URL absoluta e HTTPS
variables.put("validadeLink", "30 minutos");

String html = renderer.render("conta-criada", variables);
```

O nome passado para `render` é o nome do arquivo sem a extensão `.html`.

## Variáveis esperadas por template

Valores exibidos ao usuário devem ser textos simples. O Thymeleaf usa `th:text` para escapar conteúdo dinâmico. Envie URLs absolutas HTTPS nos campos `url...`. Formate datas e horários no backend (por exemplo, `29/10/2026 às 10:30`) antes de incluí-los no contexto.

| Template | Variáveis |
|---|---|
| `conta-criada` | `nome`, `email`, `urlDefinirSenha`, `validadeLink` (opcional) |
| `confirmar-email-login` | `nome`, `token`, `validadeMinutos` (opcional) |
| `confirmar-troca-email` | `nome`, `novoEmail`, `token`, `validadeMinutos` (opcional) |
| `confirmar-troca-senha` | `nome`, `token`, `validadeMinutos` (opcional) |
| `recuperacao-senha` | `nome`, `token`, `urlRecuperacao`, `validadeMinutos` (opcional) |
| `documento-recebido-professor` | `nomeAluno`, `tipoDocumento`, `dataHora`, `urlDocumentos` |
| `documento-avaliado-aluno` | `nome`, `tipoDocumento`, `statusAvaliacao`, `dataHora`, `urlDocumentos`, `feedback` (opcional) |
| `prazo-envio-chegando` | `nome`, `dataPrazo`, `urlAreaAluno`, `tipoDocumento` (opcional) |
| `estagio-concluido-aluno` | `nome`, `urlAreaAluno`, `dataHora` (opcional) |
| `estagio-encerrado-secretaria` | `nomeAluno`, `urlPainelSecretaria`, `dataHora` (opcional) |

Para `documento-avaliado-aluno`, passe `statusAvaliacao` como `APROVADO` ou `DESAPROVADO`. O bloco de feedback só aparece quando `feedback` não é nulo nem vazio.

## Boas práticas importantes

- **Não envie senha em texto puro.** O template de conta criada leva a um link de definição de senha. Gere um token temporário, de uso único e validade curta no backend.
- Tokens de confirmação e recuperação devem ser aleatórios, expirar, ter uso único e ser invalidados depois do uso. Evite registrar os valores dos tokens em logs.
- Não coloque dados sensíveis desnecessários no assunto do email.
- Se a URL para uma ação for opcional em algum ambiente, garanta no backend que ela seja incluída antes de renderizar o template correspondente.
- CSS de email é mantido inline de propósito; fontes externas e CSS complexo não são confiáveis em todos os clientes.
- Os links devem apontar para o domínio de produção configurado no backend, não para URLs relativas do frontend.

## Ideias de emails para considerar depois

- Senha alterada com sucesso (alerta de segurança, sem exibir a senha).
- Email alterado com sucesso.
- Documento recusado com instrução para corrigir e reenviar (caso o fluxo de avaliação exija essa comunicação separada).
- Lembrete de prazo com frequência configurável e sem envio duplicado.
