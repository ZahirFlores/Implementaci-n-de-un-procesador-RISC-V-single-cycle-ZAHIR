# Implementación de un procesador RISC-V single cycle


---

## 💻 Simulación de Ondas (QuestaSim)

Comprobación del funcionamiento del hardware ejecutando saltos dinámicos y operaciones en la ALU:

![Simulación de Ondas](Simu1.png)

## 🗺️ Datapath de la Arquitectura

El siguiente diagrama representa la ruta de datos completa implementada en el hardware para soportar la instrucción JAL:

```mermaid
graph LR
    %% Definición de Estilos
    classDef fetch fill:#e1f5fe,stroke:#0288d1,stroke-width:2px;
    classDef decode fill:#efebe9,stroke:#5d4037,stroke-width:2px;
    classDef execute fill:#e8f5e9,stroke:#388e3c,stroke-width:2px;
    classDef memory fill:#fff3e0,stroke:#f57c00,stroke-width:2px;
    classDef wb fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px;
    classDef control fill:#ffebee,stroke:#c62828,stroke-width:2px;

    %% Bloques Principales por Etapa
    subgraph Etapa_Fetch [Fetch - Búsqueda]
        PC[Program Counter]
        IM[Memoria de Instrucciones]
        Add4[Sumador PC + 4]
    end

    subgraph Etapa_Decode [Decode - Decodificación]
        CU[Unidad de Control]
        RF[Banco de Registros]
        IG[Generador de Inmediatos]
    end

    subgraph Etapa_Execute [Execute - Ejecución]
        MuxALU[Mux ALUSrc]
        ALU[ALU]
        AddJAL[Sumador PC + Imm]
    end

    subgraph Etapa_Memory [Memory - Memoria RAM]
        DM[Memoria de Datos]
    end

    subgraph Etapa_WriteBack [WriteBack - Escritura]
        MuxRes[Mux ResultSrc]
    end

    subgraph Logica_PC [Control del PC]
        MuxPC[Mux PCSrc]
    end

    %% Conexiones de la Ruta de Datos
    PC -->|PC_out| IM
    PC -->|PC_out| Add4
    PC -->|PC_out| AddJAL

    IM -->|Instrucción| CU
    IM -->|Rs1 / Rs2 / Rd| RF
    IM -->|Inmediato bruto| IG

    IG -->|ImmExt| MuxALU
    IG -->|ImmExt| AddJAL

    RF -->|RD1| ALU
    RF -->|RD2| MuxALU
    RF -->|RD2| DM
    
    MuxALU -->|SrcB| ALU

    ALU -->|ALUResult| DM
    ALU -->|ALUResult| MuxRes
    ALU -->|Zero| MuxPC

    DM -->|ReadData| MuxRes
    Add4 -->|PC + 4| MuxRes

    %% El lazo de retorno crítico del JAL
    MuxRes -->|Result / WD3| RF

    %% Lógica de actualización del PC
    Add4 -->|PC + 4| MuxPC
    AddJAL -->|PC_Target| MuxPC
    MuxPC -->|PC_in| PC

    %% Conexiones de Control
    CU -.->|RegWrite| RF
    CU -.->|ALUSrc| MuxALU
    CU -.->|ALUControl| ALU
    CU -.->|MemWrite| DM
    CU -.->|ResultSrc / Jump| MuxRes

    %% Asignación de Estilos a Clases
    class PC,IM,Add4 fetch;
    class CU,RF,IG decode;
    class MuxALU,ALU,AddJAL execute;
    class DM memory;
    class MuxRes wb;
    class MuxPC control;
