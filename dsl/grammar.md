\# Circio DSL – Grammar (v1)



Dit document beschrijft de \*\*eerste versie van de Domain-Specific Language (DSL)\*\* voor gestructureerde AI-interactie in Circio / Antigravity-achtige systemen.



De DSL wordt gebruikt om \*\*flows\*\*, \*\*context\*\*, \*\*tools\*\*, \*\*agents\*\*, \*\*invoer\*\*, \*\*stappen\*\* en \*\*veiligheidsregels\*\* te definiëren.  

Het doel is een duidelijke, leesbare taal voor ontwikkelaars en AI-agents.



---



\## 1. Doel van deze DSL



Deze DSL is ontworpen om:



\- AI-agents gestructureerde instructies te geven  

\- flows te beschrijven in een mensvriendelijke syntax  

\- definities van tools, context en policies vast te leggen  

\- stabiele en reproduceerbare AI-acties te creëren  

\- developers te helpen AI-workflows zonder programmeertaal te definiëren  



De DSL is \*\*geen programmeertaal\*\*, maar een \*\*configuratie- en orkestratietaal\*\*.



---



\## 2. Kernconcepten



\### 2.1 Flow



Een \*\*flow\*\* is een gestructureerde opdracht die een agent kan uitvoeren.



Een flow bevat:



\- \*\*goal\*\* – doel van de flow  

\- \*\*context\*\* – externe informatie en bronnen  

\- \*\*inputs\*\* – variabelen die door een gebruiker worden aangeleverd  

\- \*\*steps\*\* – de volgorde van acties  

\- \*\*constraints\*\* – veiligheidsregels  



---



\## 3. Syntax Overview (menselijke uitleg)



\### Voorbeeldstructuur van een flow:



flow "naam" {

goal: "..."

context { ... }

inputs { ... }

steps { ... }

constraints { ... }

}



markdown

Copy code



\### 3.1 Blokken



| Blok | Beschrijving |

|------|--------------|

| `goal` | Het einddoel van de flow |

| `context` | Beschikbare bronnen (repo, docs, environment, etc.) |

| `inputs` | Waarden die een gebruiker aan de flow doorgeeft |

| `agents` | Geeft aan welke agent(s) deze flow mogen uitvoeren |

| `steps` | Genummerde acties die de agent uitvoert |

| `constraints` | Regels of beperkingen (safety policies) |



---



\## 4. Ondersteunde Datatypes



| Type | Voorbeeld |

|------|-----------|

| `string` | `"tekst"` |

| `list` | `\["a", "b", "c"]` |

| `map/object` | `{ key = "value" }` |

| `integer` | `1`, `2`, `3` |

| `boolean` | `true`, `false` |



\*Strings staan altijd tussen dubbele quotes.\*



---



\## 5. Grammar (Formele definitie — BNF stijl)



> Dit is hoe een parser de DSL zou begrijpen.  

> Dit stuk is bedoeld voor ontwikkelaars / je verslag.



FLOW ::= "flow" STRING "{" FLOW\_BODY "}"

FLOW\_BODY ::= (GOAL | CONTEXT | INPUTS | AGENTS | STEPS | CONSTRAINTS)\*



GOAL ::= "goal:" STRING



CONTEXT ::= "context" "{" CONTEXT\_PAIR\* "}"

CONTEXT\_PAIR ::= IDENTIFIER "=" (STRING | LIST)



INPUTS ::= "inputs" "{" INPUT\_VAR\* "}"

INPUT\_VAR ::= IDENTIFIER


AGENTS      ::= "agents" "{" AGENT_PAIR* "}"

AGENT_PAIR  ::= IDENTIFIER ":" STRING


STEPS ::= "steps" "{" STEP\* "}"

STEP ::= INTEGER ":" IDENTIFIER



CONSTRAINTS ::= "constraints" "{" CONSTRAINT\_LINE\* "}"

CONSTRAINT\_LINE ::= "-" STRING



yaml

Copy code



---



\## 6. Voorbeeld Flow (volledig)



flow "circio\_onboarding" {



goal: "Help een nieuwe developer starten met Circio"



context {

repo = "https://github.com/project/circio"

docs = \["docs/architecture.md", "docs/flows.md"]

environment = "local"

}



inputs {

developer\_name

project\_goal

}



steps {

1: analyze\_requirements

2: propose\_agent\_setup

3: generate\_flow\_files

4: validate\_with\_tests

}



constraints {

\- "Voer geen destructieve shell-commands uit"

\- "Vraag eerst toestemming voordat bestanden worden aangepast"

}

}



yaml

Copy code



---



\## 7. Agent Definitie (aanbevolen structuur)



Agents worden in een apart YAML-bestand gedefinieerd.



```yaml

id: onboarding\_agent

name: "Circio Onboarding Assistant"



goals:

&nbsp; - "Uitleggen van DSL-structuur"

&nbsp; - "Flow-bestanden genereren"

&nbsp; - "Antwoorden valideren"



tools:

&nbsp; - name: repo\_reader

&nbsp;   type: git

&nbsp;   config:

&nbsp;     repo\_url: "https://github.com/project/circio"



&nbsp; - name: file\_writer

&nbsp;   type: filesystem

&nbsp;   config:

&nbsp;     root: "./flows"



policies:

&nbsp; - "Geen commando’s buiten projectdirectory"

&nbsp; - "Bevestiging vragen voor wijzigen van bestanden"

8\. Mogelijke uitbreidingen

Subflows



Conditionele stappen (if step 2 fails → run step 4)



Loops (repeat until success)



Multi-agent flows



Output-schema’s



Metadata \& tags



9\. Samenvatting

Deze DSL biedt:



een duidelijke flow-structuur



consistente syntax



ondersteuning voor goal, context, steps, inputs \& constraints



uitbreidbaarheid voor agents en tools



Dit document vormt de basis voor:



je verslag



parser-ontwerp



voorbeeldflows



agent-definities



toekomstige technische implementatie

