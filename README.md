# Análise da Arquitetura e Diagrama de Classes do SHA-2.0

Este documento fornece uma análise detalhada da arquitetura do software **Simulador de Hidrômetro Analógico (SHA-2.0)**, com foco na explicação do seu Diagrama de Classes UML. O objetivo é servir como um guia para entender a estrutura do código, os padrões de projeto aplicados e as responsabilidades de cada componente.

A arquitetura foi projetada para ser desacoplada e extensível, separando as responsabilidades do sistema em pacotes lógicos e utilizando padrões de projeto consagrados para gerenciar o estado e a comunicação entre os componentes.

## 🏛️ O Diagrama de Classes UML

O diagrama abaixo ilustra a estrutura estática do sistema. Ele mostra as classes, suas interfaces, atributos, métodos e os relacionamentos entre elas.

**<img width="1367" height="759" alt="image" src="https://github.com/user-attachments/assets/4f55b946-a50e-4498-9be9-1b152b44cd04" />
**
![Diagrama de Classes UML do SHA-2.0]


## 🧩 Análise dos Componentes por Pacote

A estrutura do projeto é dividida em pacotes que separam as responsabilidades, seguindo uma abordagem similar ao padrão MVC (Model-View-Controller).

### `com.meu_pacote.model` (O Coração do Sistema)
Este pacote contém a lógica de negócio principal.
* **`Hidrometro`**: É a classe central do sistema. [cite_start]Ela atua como **`<<Subject>>`** no padrão Observer e **`<<Context>>`** no padrão State[cite: 114, 115]. Suas principais responsabilidades são:
    * [cite_start]Manter o estado atual da simulação (`consumoTotalM3`, `pressaoAtualKpa`)[cite: 17, 18].
    * [cite_start]Gerenciar uma lista de observadores (`Display`) e notificá-los sobre mudanças[cite: 17, 23].
    * [cite_start]Delegar o comportamento de medição de fluxo para o objeto de estado atual (`estadoAtual.medirFluxo()`)[cite: 27].
* **`DadosHidrometro`**: É um **D**ata **T**ransfer **O**bject (`<<DTO>>`). [cite_start]Sua função é encapsular os dados do hidrômetro em um objeto imutável para ser enviado às camadas de visão e API[cite: 11]. Isso garante que a UI não possa modificar o estado do modelo diretamente.

### `com.meu_pacote.state` (Padrão de Projeto State)
[cite_start]Este pacote implementa o padrão State para gerenciar os diferentes comportamentos do hidrômetro[cite: 113, 114].
* [cite_start]**`EstadoHidrometro`**: É a interface (`<<Interface>>`) que define o contrato que todos os estados concretos devem seguir, declarando o método `medirFluxo()`[cite: 47].
* **`EstadoComAgua`**, **`EstadoSemAgua`**, **`EstadoComAr`**: São as implementações concretas do estado. [cite_start]Cada uma encapsula a lógica específica para uma condição de operação do hidrômetro (fluxo normal, sem água ou com ar)[cite: 34, 38, 48, 110, 111, 112]. [cite_start]Elas mantêm uma referência ao `Hidrometro` para acessar seus dados e para realizar a transição para um novo estado[cite: 34, 45, 52].

### `com.meu_pacote.ui` (A Camada de Visualização)
Responsável pela interface gráfica do usuário (GUI).
* [cite_start]**`Display`**: Atua como um **`<<Observer>>`** no padrão Observer[cite: 115]. Ela é responsável por renderizar o estado do hidrômetro. [cite_start]Seu método `update(DadosHidrometro)` é invocado pelo `Hidrometro` sempre que os dados mudam, garantindo que a UI esteja sempre sincronizada com o modelo[cite: 77, 115].

### `com.meu_pacote.api` (A Camada de Controle/API)
Expõe os dados do simulador para sistemas externos.
* [cite_start]**`ControladorAPI`**: Utiliza o framework **Javalin** para criar um servidor web e expor os dados do `Hidrometro` através de endpoints REST[cite: 3, 6, 112]. [cite_start]Ele possui uma referência ao `Hidrometro` para consultar os dados atuais quando uma requisição é recebida[cite: 3, 6].

### `com.meu_pacote` (Ponto de Entrada da Aplicação)
* [cite_start]**`MainApp`**: É a classe principal que inicia a aplicação[cite: 82]. [cite_start]Ela é responsável por criar as instâncias de todos os objetos principais (`Hidrometro`, `Display`, `ControladorAPI`) e conectá-los, configurando os padrões Observer e State[cite: 84, 85]. [cite_start]Ela também gerencia o loop de simulação que avança o tempo e atualiza o sistema periodicamente[cite: 86].

## 🔗 Relacionamentos e Notações Chave

* **Composição (`*--`)**: Usada para indicar que um objeto "possui" outro e gerencia seu ciclo de vida. Ex: `MainApp` é composto por um `Hidrometro`. Se a `MainApp` for encerrada, o `Hidrometro` também é.
* **Agregação (`*--`)**: Representa uma relação "tem-um" forte. Ex: `Hidrometro` tem uma lista de `Display`s. A notação é similar à composição em PlantUML, mas o contexto de ser uma coleção de observadores define a natureza do relacionamento.
* **Associação (`--`)**: Uma relação estrutural entre classes. Ex: `EstadoComAgua` tem uma associação com `Hidrometro` para poder interagir com o contexto.
* **Realização/Implementação (`<|..`)**: Indica que uma classe implementa uma interface. Ex: `EstadoComAgua` implementa `EstadoHidrometro`.
* **Dependência (`..>`)**: Indica que uma classe "usa" outra. É um relacionamento mais fraco. Ex: `Display` depende de `DadosHidrometro`, pois o recebe como parâmetro no método `update`.
* **Estereótipos (`<<...>>`)**: Rótulos usados para dar um significado semântico adicional a um elemento do diagrama, como `<<Subject>>`, `<<Observer>>`, `<<DTO>>`, para clarificar o papel da classe em um padrão de projeto.

---
Este documento reflete a arquitetura da versão 2.0 do SHA, destacando um design robusto e baseado em padrões que favorece a manutenibilidade e a expansão futura.

<details>
<summary>Clique para ver o código PlantUML que gerou este diagrama</summary>

```plantuml
@startuml
' Título do Diagrama
title Diagrama de Classes - Simulador de Hidrômetro Analógico (SHA 2.0)

' Configurações de Aparência para maior formalismo e legibilidade
skinparam linetype ortho
skinparam classAttributeIconSize 0
skinparam packageStyle rect
skinparam stereotypeCBackgroundColor #ADD8E6

' Definição da Classe Principal da Aplicação
package com.meu_pacote <<Application>> {
    class MainApp extends javafx.application.Application {
        - hidrometro: Hidrometro
        - controladorAPI: ControladorAPI
        - simulationExecutor: ScheduledExecutorService
        + {static} main(args: String[]): void
        + start(primaryStage: Stage): void
        + stop(): void
    }
}

' Pacote da Interface do Usuário (View)
package com.meu_pacote.ui <<View>> {
    class Display <<Observer>> {
        - stage: Stage
        - consumoText: Text
        - pressaoText: Text
        - estadoText: Text
        - consumoDigitalText: Text
        - ponteiroLitros: Line
        - rotLitros: Rotate
        + Display(stage: Stage)
        - initUI(): void
        + update(dados: DadosHidrometro): void
    }
}

' Pacote da API REST (Controller)
package com.meu_pacote.api <<Controller>> {
    class ControladorAPI {
        - hidrometro: Hidrometro
        - app: Javalin
        + ControladorAPI(hidrometro: Hidrometro)
        + iniciar(): void
        + parar(): void
    }
}

' Pacote do Modelo (Core Logic)
package com.meu_pacote.model <<Model>> {
    class Hidrometro <<Subject>> {
        - estadoAtual: EstadoHidrometro
        - observers: List<Display>
        - random: Random
        - consumoTotalM3: double
        - pressaoAtualKpa: double
        - pressaoBaseKpa: double
        - vazaoLPS: double
        + Hidrometro()
        + setEstado(novoEstado: EstadoHidrometro): void
        + adicionarObserver(observer: Display): void
        + notificarObservers(): void
        + simularPassagemDeTempo(deltaTime: double): void
        + getDadosAtuais(): DadosHidrometro
        + getVazaoLPS(): double
        + adicionarConsumo(volumeAdicionalM3: double): void
        + getPressaoAtualKpa(): double
    }

    class DadosHidrometro <<DTO>> {
        - consumoTotalM3: double
        - pressaoAtualKpa: double
        - estado: String
        + DadosHidrometro(consumo: double, pressao: double, estado: String)
        + getConsumoTotalM3(): double
        + getPressaoAtualKpa(): double
        + getEstado(): String
    }
}

' Pacote de Estados (State Pattern)
package com.meu_pacote.state <<State Pattern>> {
    interface EstadoHidrometro <<Interface>> {
        + {abstract} medirFluxo(deltaTime: double): void
    }

    class EstadoComAgua implements EstadoHidrometro {
        - hidrometro: Hidrometro
        + EstadoComAgua(hidrometro: Hidrometro)
        + medirFluxo(deltaTime: double): void
    }

    class EstadoSemAgua implements EstadoHidrometro {
        - hidrometro: Hidrometro
        - random: Random
        - tempoSemAgua: double
        + EstadoSemAgua(hidrometro: Hidrometro)
        + medirFluxo(deltaTime: double): void
    }

    class EstadoComAr implements EstadoHidrometro {
        - hidrometro: Hidrometro
        - random: Random
        - tempoComAr: double
        + EstadoComAr(hidrometro: Hidrometro)
        + medirFluxo(deltaTime: double): void

código do diagrama em plantUML

@startuml
' Título do Diagrama
title Diagrama de Classes - Simulador de Hidrômetro Analógico (SHA 2.0)

' Configurações de Aparência para maior formalismo e legibilidade
skinparam linetype ortho
skinparam classAttributeIconSize 0
skinparam packageStyle rect
skinparam stereotypeCBackgroundColor #ADD8E6

' Definição da Classe Principal da Aplicação
package com.meu_pacote <<Application>> {
    class MainApp extends javafx.application.Application {
        - hidrometro: Hidrometro
        - controladorAPI: ControladorAPI
        - simulationExecutor: ScheduledExecutorService
        + {static} main(args: String[]): void
        + start(primaryStage: Stage): void
        + stop(): void
    }
}

' Pacote da Interface do Usuário (View)
package com.meu_pacote.ui <<View>> {
    class Display <<Observer>> {
        - stage: Stage
        - consumoText: Text
        - pressaoText: Text
        - estadoText: Text
        - consumoDigitalText: Text
        - ponteiroLitros: Line
        - rotLitros: Rotate
        + Display(stage: Stage)
        - initUI(): void
        + update(dados: DadosHidrometro): void
    }
}

' Pacote da API REST (Controller)
package com.meu_pacote.api <<Controller>> {
    class ControladorAPI {
        - hidrometro: Hidrometro
        - app: Javalin
        + ControladorAPI(hidrometro: Hidrometro)
        + iniciar(): void
        + parar(): void
    }
}

' Pacote do Modelo (Core Logic)
package com.meu_pacote.model <<Model>> {
    class Hidrometro <<Subject>> {
        - estadoAtual: EstadoHidrometro
        - observers: List<Display>
        - random: Random
        - consumoTotalM3: double
        - pressaoAtualKpa: double
        - pressaoBaseKpa: double
        - vazaoLPS: double
        + Hidrometro()
        + setEstado(novoEstado: EstadoHidrometro): void
        + adicionarObserver(observer: Display): void
        + notificarObservers(): void
        + simularPassagemDeTempo(deltaTime: double): void
        + getDadosAtuais(): DadosHidrometro
        + getVazaoLPS(): double
        + adicionarConsumo(volumeAdicionalM3: double): void
        + getPressaoAtualKpa(): double
    }

    class DadosHidrometro <<DTO>> {
        - consumoTotalM3: double
        - pressaoAtualKpa: double
        - estado: String
        + DadosHidrometro(consumo: double, pressao: double, estado: String)
        + getConsumoTotalM3(): double
        + getPressaoAtualKpa(): double
        + getEstado(): String
    }
}

' Pacote de Estados (State Pattern)
package com.meu_pacote.state <<State Pattern>> {
    interface EstadoHidrometro <<Interface>> {
        + {abstract} medirFluxo(deltaTime: double): void
    }

    class EstadoComAgua implements EstadoHidrometro {
        - hidrometro: Hidrometro
        + EstadoComAgua(hidrometro: Hidrometro)
        + medirFluxo(deltaTime: double): void
    }

    class EstadoSemAgua implements EstadoHidrometro {
        - hidrometro: Hidrometro
        - random: Random
        - tempoSemAgua: double
        + EstadoSemAgua(hidrometro: Hidrometro)
        + medirFluxo(deltaTime: double): void
    }

    class EstadoComAr implements EstadoHidrometro {
        - hidrometro: Hidrometro
        - random: Random
        - tempoComAr: double
        + EstadoComAr(hidrometro: Hidrometro)
        + medirFluxo(deltaTime: double): void
    }
}

' --- RELACIONAMENTOS ---

' Relacionamentos da Classe Principal (MainApp)
MainApp *-- "1" Hidrometro : cria e gerencia
MainApp *-- "1" ControladorAPI : cria e gerencia
MainApp ..> Display : instancia

' Padrão Observer: Hidrometro (Subject) notifica Display (Observer)
Hidrometro "1" *-- "0..*" Display : observers
note right on link : O Hidrometro (Subject) mantém uma\nlista de Displays (Observers) e os notifica\nchamando o método update(). [cite: 17, 22, 23, 24]

' Relacionamentos do Controlador da API
ControladorAPI -- "1" Hidrometro : referencia

' Relacionamentos de Dependência para o DTO
Hidrometro ..> DadosHidrometro : instancia e retorna
Display ..> DadosHidrometro : recebe como parâmetro
ControladorAPI ..> DadosHidrometro : usa através do Hidrometro

' Padrão State: Hidrometro (Context) delega para EstadoHidrometro (State)
Hidrometro "1" *-- "1" EstadoHidrometro : estadoAtual
note right on link
  O Hidrometro (Context) possui uma instância
  de um estado concreto e delega a execução
  do método medirFluxo() para ela. [cite: 16, 27]
end note

EstadoHidrometro <|.. EstadoComAgua
EstadoHidrometro <|.. EstadoSemAgua
EstadoHidrometro <|.. EstadoComAr

' Associações dos estados concretos com o Contexto (Hidrometro)
EstadoComAgua -- "1" Hidrometro : referencia o contexto
EstadoSemAgua -- "1" Hidrometro : referencia o contexto
EstadoComAr -- "1" Hidrometro : referencia o contexto

' Dependências entre estados para transição
EstadoSemAgua ..> EstadoComAr : instancia para transição 
EstadoComAr ..> EstadoComAgua : instancia para transição [cite: 45]

' Nota sobre classes omitidas para clareza
note "A classe org.example.Main foi omitida por ser código boilerplate gerado pela IDE e não fazer parte da arquitetura do simulador. [cite: 93]\nAs classes da biblioteca JavaFX (Stage, Text, etc.) e Javalin são representadas como tipos, mas não detalhadas no diagrama." as N1
@enduml
