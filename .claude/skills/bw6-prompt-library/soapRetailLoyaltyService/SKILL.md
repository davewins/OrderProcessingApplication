---
name: soapRetailLoyaltyService
description: Build the "retailLoyaltyService" BW6 application — a SOAP service generated from `LoyaltyService.wsdl` implementing `GetLoyaltyPoints`, with auto-generated `SOAP Receive`/`SOAP Reply` plus Mapper and Log in between. Use when the user asks to create/scaffold the retail loyalty SOAP service, needs a BW6 SOAP palette sample generated FROM a WSDL, or references any of: "retail loyalty", "retailLoyaltyService", "LoyaltyService.wsdl", "GetLoyaltyPoints", "SOAP Receive", "SOAP Reply", "Generate Process from WSDL", "LoyaltyPortType", "loyalty points service". Drives the build via `bwdesign` / `mcp__bw__*` tools and applies `bw6-rules` checks.
---

# retailLoyaltyService — SOAP Palette Sample (BW6)

Introduces the **SOAP-from-WSDL** flow: import a WSDL, generate a process from it (auto-creating `SOAP Receive` and `SOAP Reply`), then wire the middle activities.

Category: **SOAP** • Main tech: `WSDL, SOAP Receive, SOAP Reply`.

## How to run this skill

1. Confirm the target BW workspace with the user. Prefer `mcp__bw__*` if Business Studio is open, else `bwdesign` per the `bwdesign` skill.
2. **Critical constraint:** the `SOAP Receive` must be generated from the WSDL — do NOT create it manually. Use **Generate Process from WSDL** (Studio) or the corresponding MCP tool.
3. Execute the spec below step by step. Announce each major step.
4. Cross-check against `bw6-rules`. Rules to watch here:
   - `BindingShouldHavePolicyAssociated` / `BindingShouldNotHaveHTTPBasicPolicyAssociated` — SOAP binding needs a non-Basic auth policy for anything beyond dev.
   - `EndpointURIFromHTTPBindingSetUsingProperty` — bind the SOAP endpoint URI to a Module Property.
   - `HttpConnectorShouldHaveConfidentiality` — enable SSL on the HTTP Connector if targeting HTTPS.
   - `DefaultTargetNamespace` — the WSDL uses a real namespace; keep it.
   - `LastActivityAndEndActivity` — the SOAP Reply terminates the flow correctly.
5. Validate and report status.

## Project Specification

### Project Hierarchy

| Component Type | Name |
| :---- | :---- |
| **Application Module** | `retailLoyaltyService` |
| **Application Project** | `retailLoyaltyService.application` |

### WSDL Setup

1. Create `C:\Retail\WSDL\LoyaltyService.wsdl` with the contents below.
2. In Business Studio, right-click the `retailLoyaltyService` module → **Import → File System** → browse to `C:\Retail\WSDL\` → import `LoyaltyService.wsdl`.
3. Verify the WSDL appears under project resources before proceeding.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<wsdl:definitions xmlns:wsdl="http://schemas.xmlsoap.org/wsdl/"
                  xmlns:soap="http://schemas.xmlsoap.org/wsdl/soap/"
                  xmlns:tns="http://www.tibco.com/retail/loyalty"
                  xmlns:xs="http://www.w3.org/2001/XMLSchema"
                  targetNamespace="http://www.tibco.com/retail/loyalty"
                  name="LoyaltyService">

    <wsdl:types>
        <xs:schema targetNamespace="http://www.tibco.com/retail/loyalty">
            <xs:element name="LoyaltyRequest">
                <xs:complexType>
                    <xs:sequence>
                        <xs:element name="customerId"  type="xs:string"/>
                        <xs:element name="requestDate" type="xs:date"/>
                    </xs:sequence>
                </xs:complexType>
            </xs:element>
            <xs:element name="LoyaltyResponse">
                <xs:complexType>
                    <xs:sequence>
                        <xs:element name="customerId"    type="xs:string"/>
                        <xs:element name="loyaltyPoints" type="xs:integer"/>
                        <xs:element name="customerTier"  type="xs:string"/>
                    </xs:sequence>
                </xs:complexType>
            </xs:element>
        </xs:schema>
    </wsdl:types>

    <wsdl:message name="LoyaltyRequestMessage">
        <wsdl:part name="parameters" element="tns:LoyaltyRequest"/>
    </wsdl:message>

    <wsdl:message name="LoyaltyResponseMessage">
        <wsdl:part name="parameters" element="tns:LoyaltyResponse"/>
    </wsdl:message>

    <wsdl:portType name="LoyaltyPortType">
        <wsdl:operation name="GetLoyaltyPoints">
            <wsdl:input  message="tns:LoyaltyRequestMessage"/>
            <wsdl:output message="tns:LoyaltyResponseMessage"/>
        </wsdl:operation>
    </wsdl:portType>

    <wsdl:binding name="LoyaltyBinding" type="tns:LoyaltyPortType">
        <soap:binding style="document"
                      transport="http://schemas.xmlsoap.org/soap/http"/>
        <wsdl:operation name="GetLoyaltyPoints">
            <soap:operation soapAction="GetLoyaltyPoints"/>
            <wsdl:input>  <soap:body use="literal"/> </wsdl:input>
            <wsdl:output> <soap:body use="literal"/> </wsdl:output>
        </wsdl:operation>
    </wsdl:binding>

    <wsdl:service name="LoyaltyPointsService">
        <wsdl:port name="LoyaltyPort" binding="tns:LoyaltyBinding">
            <soap:address location="http://localhost:8080/loyalty"/>
        </wsdl:port>
    </wsdl:service>

</wsdl:definitions>
```

### Process `LoyaltyProcess.bwp`

1. Right-click package `retailLoyaltyService` → **Generate Process from WSDL**.
2. Source WSDL: `LoyaltyService.wsdl`.
3. Operation: `GetLoyaltyPoints`.
4. This auto-generates `SOAP Receive` and `SOAP Reply` with correct schema bindings.

Between the auto-generated activities, add and link in sequence: `Mapper` → `Log`.

- **SOAP Receive** — auto-generated; do not modify.
- **Mapper**
  - `customerId` ← `$SOAPReceive/Body/tns:LoyaltyRequest/tns:customerId`
  - `loyaltyPoints` ← `xsd:integer("250")`
  - `customerTier` ← `"Gold"`
- **Log** — `message` = `concat("Retail loyalty request received for customer: ", $SOAPReceive/Body/tns:LoyaltyRequest/tns:customerId)`
- **SOAP Reply** (auto-generated)
  - `customerId` ← `$Mapper/customerId`
  - `loyaltyPoints` ← `$Mapper/loyaltyPoints`
  - `customerTier` ← `$Mapper/customerTier`
