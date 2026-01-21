# OR4 – DSL Design (Working Notes)

## Doel
Een Domain-Specific Language (DSL) ontwerpen waarmee:
- agents en flows gestructureerd beschreven worden
- inputs/outputs expliciet getypeerd zijn (TMAP concept)
- nodes/functies semantisch benoemd zijn (FMAP concept)
- de structuur direct bruikbaar is voor visualisatie (OR1)

## DSL eigenschappen
- Beschrijvend (declarative), niet uitvoerend
- Menselijk leesbaar + machine parsebaar
- Consistent naming (ids, types, node types)

## Flow DSL (v0.1)
Een flow bestaat uit:
- inputs (typed)
- nodes (function/api_call met input/output + types)
- connections (dataflow tussen nodes)
- outputs (typed)
- error_handling (basis foutpad)
- state_model (node state set voor OR1)

## Agent DSL (v0.1)
Een agent beschrijft:
- purpose/role/domain
- inputs/outputs (typed)
- capabilities
- constraints (basis validatieregels)

## Acceptatiecriteria (OR4)
- Elke flow heeft typed inputs en outputs
- Nodes hebben consistente types (function, api_call, ...)
- DSL structuur is stabiel genoeg om op te bouwen (visualisatie & latere validatie)


# OR4 – Correctness & Consistency in Circio DSL

Dit hoofdstuk beschrijft hoe de Circio DSL correctness (OR4) afdwingt binnen flow-definities.
Correctness betekent dat een flow semantisch en structureel valide is voordat uitvoering
of visualisatie plaatsvindt.

De OR4-regels richten zich op:
- Type-consistentie (TMAP)
- Correcte data- en control-flow
- Geldige referenties tussen nodes
- Expliciete en geldige error-afhandeling

---

## OR4-regels (samenvatting)

### OR4.1 Type consistency
- Elke `input` en `output` van een node heeft een expliciet type
- Verbonden poorten moeten type-compatibel zijn
- Een node mag geen datatype ontvangen dat niet overeenkomt met zijn `input_type`

### OR4.2 Geldige node-referenties
- Elke `connection.from` en `connection.to` verwijst naar een bestaande node
- `goto` in `router` en `error_handling` verwijst altijd naar een bestaande node

### OR4.3 Consistente dataflow
- Data mag alleen worden doorgegeven via expliciet gedefinieerde outputs
- Impliciete of “magische” outputs zijn niet toegestaan

### OR4.4 Afgesloten control-flow
- Elke mogelijke route in een `router` eindigt in een geldige node
- Fouten worden expliciet afgehandeld via `error_handling` of `terminal` nodes

---

## OR4 Demonstratie – Foute flow

Bestand: `flows/or4_invalid_example.yaml`

Deze flow is bewust incorrect opgezet om OR4-validatie te demonstreren.

### Vastgestelde fouten:

1. **Type mismatch**
   - `create_account` ontvangt `ValidationResult`
   - Verwacht wordt `UserProfile`

2. **Ongeldige connection**
   - Verwijzing naar `notify_admin`, een node die niet bestaat

3. **Ongeldige error-afhandeling**
   - `goto: non_existing_terminal` verwijst naar een niet-bestaande node

Deze flow is **niet uitvoerbaar** en zou door een Circio-validatielaag worden afgekeurd.

---

## OR4 Demonstratie – Correcte flow

Bestand: `flows/or4_valid_example.yaml`

In deze versie zijn alle OR4-fouten hersteld:

- Type-consistente inputs en outputs
- Alleen geldige node-referenties
- Correcte router- en error-afhandeling
- Expliciet afgesloten control-flow via `terminal` node

Deze flow voldoet volledig aan OR4 en is geschikt voor
uitvoering, analyse en visualisatie.

---

## Conclusie

Door OR4-regels expliciet op te nemen in de DSL-definitie wordt voorkomen dat:
- flows op runtime falen
- visualisaties onbetrouwbaar worden
- impliciete aannames in dataflow ontstaan

OR4 vormt daarmee een essentieel fundament voor robuuste
AI-gestuurde flows binnen Circio.
