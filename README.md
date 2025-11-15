# Projeto: Revisão de Código — Caixa Branca


## 1) Notação de Grafo de Fluxo (método `verificarUsuario`)

Nós (N):

```
N1: Início do método verificarUsuario(login, senha)
N2: Chamar conectarBD()
N3: Montar SQL (PreparedStatement)
N4: Entrar em try-with-resources
N5: Preparar PreparedStatement
N6: Executar query -> obter ResultSet
N7: if (rs.next())
N8: Atribuir nome; result = true
N9: fim do try
N10: return result (fim)
```

Arestas (fluxo):

```
N1→N2
N2→N3
N3→N4
N4→N5
N5→N6
N6→N7
N7→N8  (quando true)
N7→N9  (quando false)
N8→N10
N9→N10
```

Representação (ASCII):

```
 [N1] --> [N2] --> [N3] --> [N4] --> [N5] --> [N6] --> [N7] -->|true| [N8] --> [N10]
                                             |false|--> [N9] --> [N10]
```

---

## 2) Complexidade Ciclomática

Fórmula: V(G) = E - N + 2

* N (nós) = 10 (N1..N10)
* E (arestas) = 10 (conforme listadas acima)

Cálculo:

```
V(G) = 10 - 10 + 2 = 2
```

**Complexidade ciclomática = 2**

Interpretação: existem 2 caminhos linearmente independentes (geralmente: condição do `if (rs.next())` true/false).

---

## 3) Caminhos Básicos

Como V(G) = 2, apresentamos os dois caminhos básicos:

**Caminho 1 (condição verdadeira)**

```
N1 -> N2 -> N3 -> N4 -> N5 -> N6 -> N7(true) -> N8 -> N10
```

**Caminho 2 (condição falsa)**

```
N1 -> N2 -> N3 -> N4 -> N5 -> N6 -> N7(false) -> N9 -> N10
```

---

## 4) Código-fonte revisado e comentado

> Observações gerais aplicadas na refatoração:
>
> * Uso de `PreparedStatement` para evitar SQL injection;
> * `try-with-resources` para garantir fechamento automático de `Connection`, `PreparedStatement` e `ResultSet`;
> * Separação de responsabilidades: `ConnectionFactory` para abrir conexões;
> * Tratamento adequado de exceções: log e rethrow quando necessário;
> * Nomes de variáveis mais descritivos.

```java
// src/main/java/login/ConnectionFactory.java
package login;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public final class ConnectionFactory {
    private static final String URL = "jdbc:mysql://127.0.0.1:3306/test?useSSL=false";
    private static final String USER = "lopes";
    private static final String PASS = "123";

    private ConnectionFactory() {}

    public static Connection getConnection() throws SQLException {
        // O driver moderno do MySQL registra-se automaticamente. Se precisar, descomente a linha abaixo.
        // Class.forName("com.mysql.cj.jdbc.Driver");
        return DriverManager.getConnection(URL, USER, PASS);
    }
}

// src/main/java/login/UserDAO.java
package login;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

/**
 * Classe responsável pelas operações de persistência relacionadas ao usuário.
 */
public class UserDAO {

    /**
     * Verifica se existe usuário com login e senha informados.
     * @param login nome de login do usuário
     * @param senha senha em texto simples (idealmente usar hash em produção)
     * @return true se o usuário existe e as credenciais batem
     * @throws SQLException em caso de erro na camada de acesso a dados
     */
    public boolean verificarUsuario(String login, String senha) throws SQLException {
        String sql = "SELECT nome FROM usuarios WHERE login = ? AND senha = ?";
        // try-with-resources garante o fechamento automático dos recursos
        try (Connection conn = ConnectionFactory.getConnection();
             PreparedStatement ps = conn.prepareStatement(sql)) {

            ps.setString(1, login);
            ps.setString(2, senha);

            try (ResultSet rs = ps.executeQuery()) {
                if (rs.next()) {
                    String nomeEncontrado = rs.getString("nome");
                    // você pode armazenar o nome encontrado em um campo se necessário
                    return true;
                } else {
                    return false;
                }
            }
        }
    }
}
```


5) Cálculos (detalhados) — Complexidade e Caminhos

Nós (N) identificados = 10

Arestas (E) identificadas = 10

V(G) = E - N + 2 = 10 - 10 + 2 = 2

Caminhos básicos = 2 (cobertura das duas saídas do if (rs.next()))

