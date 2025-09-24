# Análise da Arquitetura e Diagrama de Classes do SHA-2.0

Este documento fornece uma análise detalhada da arquitetura do software **Simulador de Hidrômetro Analógico (SHA-2.0)**, com foco na explicação do seu Diagrama de Classes UML. O objetivo é servir como um guia para entender a estrutura do código, os padrões de projeto aplicados e as responsabilidades de cada componente.

A arquitetura foi projetada para ser desacoplada e extensível, separando as responsabilidades do sistema em pacotes lógicos e utilizando padrões de projeto consagrados para gerenciar o estado e a comunicação entre os componentes.

## 🏛️ O Diagrama de Classes UML

O diagrama abaixo ilustra a estrutura estática do sistema. Ele mostra as classes, suas interfaces, atributos, métodos e os relacionamentos entre elas.

**<img width="1367" height="759" alt="image" src="https://github.com/user-attachments/assets/4f55b946-a50e-4498-9be9-1b152b44cd04" />
**
![Diagrama de Classes UML do SHA-2.0](URL_DA_SUA_IMAGEM_AQUI.png)

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
