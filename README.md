<img width="1366" height="780" alt="image" src="https://github.com/user-attachments/assets/dafb8d26-7d70-46bc-9ed5-ac9204d113f1" />

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

diagrama 2: 
## 🏛️ Arquitetura e Diagrama de Classes (UML)

A arquitetura do projeto foi estruturada com base em princípios de design SOLID e padrões de projeto para garantir um código desacoplado, manutenível e extensível. Os dois padrões centrais utilizados são:

* [cite_start]**Padrão de Projeto State (Estado):** Utilizado para gerenciar o comportamento complexo do hidrômetro[cite: 1144]. [cite_start]Cada estado (`ComAgua`, `SemAgua`, `ComAr`) é encapsulado em sua própria classe, eliminando a necessidade de condicionais complexas e aderindo ao Princípio Aberto/Fechado[cite: 1145].
* [cite_start]**Padrão de Projeto Observer (Observador):** Usado para desacoplar o modelo (`Hidrometro`) da visão (`Display`)[cite: 1145]. [cite_start]O `Hidrometro` (o *Subject*) notifica o `Display` (o *Observer*) sobre qualquer mudança em seu estado, que então atualiza a interface gráfica sem que haja um acoplamento direto entre eles[cite: 1146].

O Diagrama de Classes a seguir, gerado com PlantUML, representa formalmente a estrutura e os relacionamentos do sistema SHA 2.0.

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
    }
}

' --- RELACIONAMENTOS ---

' Relacionamentos da Classe Principal (MainApp)
MainApp *-- "1" Hidrometro : cria e gerencia
MainApp *-- "1" ControladorAPI : cria e gerencia
MainApp ..> Display : instancia

' Padrão Observer: Hidrometro (Subject) notifica Display (Observer)
Hidrometro "1" *-- "0..*" Display : observers
note right on link : O Hidrometro (Subject) mantém uma\nlista de Displays (Observers) e os notifica\nchamando o método update().

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
  do método medirFluxo() para ela.
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
EstadoComAr ..> EstadoComAgua : instancia para transição

' Nota sobre classes omitidas para clareza
note "A classe org.example.Main foi omitida por ser código boilerplate gerado pela IDE e não fazer parte da arquitetura do simulador.\nAs classes da biblioteca JavaFX (Stage, Text, etc.) e Javalin são representadas como tipos, mas não detalhadas no diagrama." as N1
@enduml
@enduml
