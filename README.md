# API BNI Guanajuato - Documentación Técnica

**Versión:** 1.0.1  
**Fecha:** 02 de Diciembre, 2025  
**Módulo:** extendrix_bni_endpoints v16.0  
**Compatibilidad:** Odoo 16.0 + cfdi_invoice (BNI)

---

## 📑 Contenido

1. [Información General](#información-general)
2. [Autenticación](#autenticación)
3. [Endpoint: Facturas](#endpoint-facturas)
5. [Códigos de Respuesta](#códigos-de-respuesta)
6. [Referencia de Campos](#referencia-de-campos)
7. [Catálogos SAT](#catálogos-sat)


---

## Información General

### Base URL

```
Producción:  https://miembros.bnibajio.com/api/bni
```

### Formato de Respuesta

Todas las respuestas son JSON con esta estructura:

```json
{
  "success": true,
  "data": {},
  "response_time": 0.234,
  "timestamp": "2025-11-20T10:00:00"
}
```

### Headers Requeridos

```
Content-Type: application/json
Authorization: Bearer {access_token}
```

---

## Autenticación

### 🔑 Generar Token de Acceso

**Endpoint:** `POST /api/bni/auth/token`

**Descripción:** Genera un token de acceso para autenticar las peticiones a la API.

#### Ejemplo en Postman

**Request:**

```
Method: POST
URL: https://miembros.bnibajio.com/api/bni/auth/token
Headers:
  Content-Type: application/json

Body (raw JSON):
{
  "username": "usuario@bni-guanajuato.com",
  "password": "contraseña_segura",
  "expires_in": 3600
}
```

**Response (200 OK):**

```json
{
  "jsonrpc": "2.0",
  "id": null,
  "result": {
    "success": true,
    "access_token": "3ed4e49c7657399131209001ef62fddf0d08125a1d58b61668aac8ca1c55193e",
    "token_type": "Bearer",
    "expires_in": 3600,
    "created_at": "2025-11-24T12:17:36",
    "expires_at": "2025-11-24T13:17:36",
    "user": {
      "id": 2,
      "name": "Administrator",
      "email": "support@extendrix.com"
    },
    "response_time": 1.731,
    "timestamp": "2025-11-24T12:17:36.876382"
  }
}
```

**Errores:**

```json
// 401 - Credenciales inválidas
{
  "jsonrpc": "2.0",
  "id": null,
  "result": {
    "success": false,
    "error": "Credenciales inválidas",
    "code": "INVALID_CREDENTIALS",
    "response_time": 1.35,
    "timestamp": "2025-11-24T12:18:31.493178"
  }
}

// 403 - Sin permisos
{
  "jsonrpc": "2.0",
  "id": null,
  "result": {
    "success": false,
    "error": "Este usuario no tiene acceso habilitado a la API",
    "code": "API_ACCESS_DISABLED",
    "response_time": 1.424,
    "timestamp": "2025-11-24T12:20:28.110596"
  }
}
```

#### Parámetros

| Campo      | Tipo    | Requerido | Descripción                      | Default |
|------------|---------|-----------|----------------------------------|---------|
| username   | string  | ✅ Sí      | Email del usuario Odoo           | -       |
| password   | string  | ✅ Sí      | Contraseña del usuario           | -       |
| expires_in | integer | ❌ No      | Tiempo de expiración en segundos | 3600    |

---

### 🔍 Validar Token

**Endpoint:** `POST /api/bni/auth/validate`

**Descripción:** Valida si un token es válido y retorna información sobre su expiración.

#### Ejemplo en Postman

**Request:**

```
Method: POST
URL: https://miembros.bnibajio.com/api/bni/auth/validate
Headers:
  Content-Type: application/json

Body (raw JSON):
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Response (200 OK):**

```json
{
  "success": true,
  "valid": true,
  "token_info": {
    "user_id": 2,
    "user_name": "Juan Pérez",
    "created_at": "2025-11-20T10:00:00",
    "expires_at": "2025-11-20T11:00:00",
    "remaining_seconds": 2847
  },
  "response_time": 0.045,
  "timestamp": "2025-11-20T10:12:33"
}
```

---

## Endpoint: Facturas

### 📋 Consultar Facturas

**Endpoint:** `GET /api/bni/facturas`

**Descripción:** Consulta facturas emitidas con filtros opcionales.

#### Parámetros de Consulta

| Parámetro    | Tipo    | Requerido | Descripción                   | Ejemplo          |
|--------------|---------|-----------|-------------------------------|------------------|
| fecha_desde  | string  | ❌ No      | Fecha inicial (YYYY-MM-DD)    | 2025-01-01       |
| fecha_hasta  | string  | ❌ No      | Fecha final (YYYY-MM-DD)      | 2025-12-31       |
| uuid         | string  | ❌ No      | Folio fiscal (UUID)           | A1B2C3D4-E5F6... |
| folio        | string  | ❌ No      | Folio de factura              | 20250145         |
| serie        | string  | ❌ No      | Serie de factura              | BNI              |
| rfc_receptor | string  | ❌ No      | RFC del cliente               | BNA020202BBB     |
| estado       | string  | ❌ No      | Estado de la factura          | vigente          |
| limit        | integer | ❌ No      | Máximo de resultados (1-1000) | 100              |
| offset       | integer | ❌ No      | Desplazamiento                | 0                |

#### Estados Válidos

| Código                | Descripción               |
|-----------------------|---------------------------|
| vigente               | Factura correcta y activa |
| cancelada             | Factura cancelada         |
| cancelacion_proceso   | Cancelación en proceso    |
| cancelacion_rechazada | Cancelación rechazada     |

#### Ejemplo en Postman

**Request:**

```
Method: GET
URL: https://miembros.bnibajio.com/api/bni/facturas?fecha_desde=2025-11-20&fecha_hasta=2025-11-20&limit=1
Headers:
  Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Response (200 OK):**

```json
{
    "success": true,
    "total": 1,
    "count": 1,
    "limit": 1,
    "offset": 0,
    "has_more": false,
    "facturas": [
        {
            "id": 38,
            "name": "INV/2025/00005",
            "folio": "202500005",
            "uuid": "6fb83423-3804-43d4-89ef-2238ba35bbb4",
            "serie": "FN",
            "fecha_emision": "2025-11-20T04:43:54.936881",
            "fecha_certificacion": "2025-11-19T22:43:57",
            "rfc_emisor": "BME0603237W7",
            "nombre_emisor": "REFERENCIAS Y NETWORKING",
            "regimen_fiscal_emisor": "601",
            "rfc_receptor": "SVB100527260",
            "nombre_receptor": "SOCIOS EN VENTAS BAJIO",
            "codigo_postal_receptor": "38050",
            "regimen_fiscal_receptor": "601",
            "residencia_fiscal": "",
            "num_reg_id_trib": "",
            "subtotal": 3728.45,
            "descuento": 0.0,
            "impuestos": 596.55,
            "total": 4325.0,
            "moneda": "MXN",
            "tipo_cambio": "1",
            "uso_cfdi": "G03",
            "uso_cfdi_descripcion": "G03",
            "metodo_pago": "PUE",
            "forma_pago": "03",
            "forma_pago_descripcion": "03",
            "tipo_comprobante": "I",
            "lugar_expedicion": "11510",
            "exportacion": "01",
            "confirmacion": "",
            "fac_atr_adquirente": "",
            "estatus_factura": "vigente",
            "estado_sat": "factura_correcta",
            "factura_cfdi": true,
            "proceso_timbrado": false,
            "numero_certificado": "00001000000710191149",
            "numero_certificado_sat": "00001000000719545303",
            "sello_digital_cfdi": "qa3XgX30pMMd8RU2j+w9Wm2vI5almNT+nLyYYzAeOv+JJQSnf7O4duDoWhDy7JF3SyosazcY5n6KP18UxLgY2ouA8YCOq0fZJkE1zEirxSLsPGUibJ77DaBRC17f8uk+0olDkJ7CDkwHyq5f5bsk7Tpt7/NtdSMbIORJc9OJsFDWrGFiQhmC6Qj1rKS8lQ8gN6yN5tofV4qC/0Mg45uCu4WSL8akEw//RmK508KNdK+btHg9gKJL7mFW5+uuCjxNdKy76GHe1lZAksLE0np2pQGBfaLCU/aso5DvzXANBiA3ry6+gS4RpfSPn4hD3oni33RcdKIrY/U7IV8esra2xQ==",
            "sello_sat": "I9qQovVORvwvrJ9xq8N63YweWmJQK+8yYjPey9BZtZuTARhtz/ucIteHhUcGQjQZOML0vUAwIhoDw5yNZnK9GPxQjuez46nxKGJ5JwxD9rhpY/L++ipqvhpEDf99HpeLLSM46aZu8G/2alWz0I73le8gdPqZZ8rToJfjr3qhMoymbpi57jcgYEOfyrX8nj2y8eA2+rZtq9CbACGWt9LY6GnJWNgb9H7wcLjFqcibCrxDCJyOiBX/JkgX0KXc4XOVD3etkBYviTmrc2Yi4UUml3UQNSheuFNSQCg6F4lDjJhFbIj22GhmhJ22EvtsJKfW2xXrp90b5q3cL7R8ZWJBnQ==",
            "cadena_original": "||1.1|6fb83423-3804-43d4-89ef-2238ba35bbb4|2025-11-19T22:43:57|qa3XgX30pMMd8RU2j+w9Wm2vI5almNT+nLyYYzAeOv+JJQSnf7O4duDoWhDy7JF3SyosazcY5n6KP18UxLgY2ouA8YCOq0fZJkE1zEirxSLsPGUibJ77DaBRC17f8uk+0olDkJ7CDkwHyq5f5bsk7Tpt7/NtdSMbIORJc9OJsFDWrGFiQhmC6Qj1rKS8lQ8gN6yN5tofV4qC/0Mg45uCu4WSL8akEw//RmK508KNdK+btHg9gKJL7mFW5+uuCjxNdKy76GHe1lZAksLE0np2pQGBfaLCU/aso5DvzXANBiA3ry6+gS4RpfSPn4hD3oni33RcdKIrY/U7IV8esra2xQ==|00001000000719545303||",
            "qr_value": "https://verificacfdi.facturaelectronica.sat.gob.mx/default.aspx?&id=6fb83423-3804-43d4-89ef-2238ba35bbb4&re=BME0603237W7&rr=SVB100527260&tt=0000004325.000000&fe=sra2xQ==",
            "uuid_relacionado": "",
            "tipo_relacion": "",
            "factura_global": false,
            "fg_periodicidad": "",
            "fg_meses": "",
            "fg_ano": "",
            "fecha_vencimiento": "2025-11-19",
            "invoice_date": "2025-11-19",
            "state": "posted",
            "move_type": "out_invoice",
            "amount_to_text": "CUATRO MIL TRESCIENTOS VEINTICINCO PESOS 00/100 M. N.",
            "amount_residual": 0.0,
            "ordenes_venta": [
                {
                    "name": "VN00039",
                    "cliente": "Raul García Granados Gudiño"
                }
            ],
            "conceptos": [
                {
                    "id": 97,
                    "numero_linea": 0,
                    "clave_sat": "80141902",
                    "clave_unidad": "E48",
                    "unidad_medida": "Unidad de servicio",
                    "no_identificacion": "",
                    "descripcion": "Membresía por 12 meses",
                    "cantidad": 1.0,
                    "precio_unitario": 4325.0,
                    "importe": 3728.45,
                    "descuento": 0.0,
                    "valor_unitario": 4325.0,
                    "objeto_imp": "02",
                    "pedimento": "",
                    "predial": "",
                    "impuestos": {
                        "traslados": [
                            {
                                "nombre": "IVA(16%) VENTAS",
                                "impuesto": "002",
                                "tipo_factor": "Tasa",
                                "base": 3728.45,
                                "tasa_o_cuota": 0.16,
                                "importe": 596.552
                            }
                        ],
                        "total_traslados": 596.552
                    }
                }
            ],
            "resumen_impuestos": {
                "translados": [
                    {
                        "impuesto": "002",
                        "TipoFactor": "Tasa",
                        "tasa": "0.160000",
                        "importe": 596.55,
                        "base": 3728.45,
                        "tax_id": 2
                    }
                ],
                "TotalImpuestosTrasladados": "596.55"
            },
            "pagos_relacionados": [
                {
                    "diario": "STP",
                    "fecha": "2025-11-20",
                    "monto": 4325.0,
                    "metodo_pago": "STP"
                }
            ],
            "numero_pagos": 1,
            "xml_base64": "PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0idXRmLTgiPz48Y2ZkaTpDb21wcm9iYW50ZSB4bWxuczpjZmRpPSJodHRwOi8vd3d3LnNhdC5nb2IubXgvY2ZkLzQiIHhtbG5zOnhzaT0iaHR0cDovL3d3dy53My5vcmcvMjAwMS9YTUxTY2hlbWEtaW5zdGFuY2UiIHhzaTpzY2hlbWFMb2NhdGlvbj0iaHR0cDovL3d3dy5zYXQuZ29iLm14L2NmZC80IGh0dHA6Ly93d3cuc2F0LmdvYi5teC9zaXRpb19pbnRlcm5ldC9jZmQvNC9jZmR2NDAueHNkIiBWZXJzaW9uPSI0LjAiIFNlcmllPSJGTiIgRm9saW89IjIwMjUwMDAwNSIgRmVjaGE9IjIwMjUtMTEtMTlUMjI6NDM6NTQiIEZvcm1hUGFnbz0iMDMiIE5vQ2VydGlmaWNhZG89IjAwMDAxMDAwMDAwNzEwMTkxMTQ5IiBTdWJUb3RhbD0iMzcyOC40NSIgRGVzY3VlbnRvPSIwIiBNb25lZGE9Ik1YTiIgVGlwb0NhbWJpbz0iMSIgVG90YWw9IjQzMjUiIFRpcG9EZUNvbXByb2JhbnRlPSJJIiBFeHBvcnRhY2lvbj0iMDEiIE1ldG9kb1BhZ289IlBVRSIgTHVnYXJFeHBlZGljaW9uPSIxMTUxMCIgQ2VydGlmaWNhZG89Ik1JSUdIekNDQkFlZ0F3SUJBZ0lVTURBd01ERXdNREF3TURBM01UQXhPVEV4TkRrd0RRWUpLb1pJaHZjTkFRRUxCUUF3Z2dHVk1UVXdNd1lEVlFRRERDeEJReUJFUlV3Z1UwVlNWa2xEU1U4Z1JFVWdRVVJOU1U1SlUxUlNRVU5KVDA0Z1ZGSkpRbFZVUVZKSlFURXVNQ3dHQTFVRUNnd2xVMFZTVmtsRFNVOGdSRVVnUVVSTlNVNUpVMVJTUVVOSlQwNGdWRkpKUWxWVVFWSkpRVEVhTUJnR0ExVUVDd3dSVTBGVUxVbEZVeUJCZFhSb2IzSnBkSGt4TWpBd0Jna3Foa2lHOXcwQkNRRVdJM05sY25acFkybHZjMkZzWTI5dWRISnBZblY1Wlc1MFpVQnpZWFF1WjI5aUxtMTRNU1l3SkFZRFZRUUpEQjFCZGk0Z1NHbGtZV3huYnlBM055d2dRMjlzTGlCSGRXVnljbVZ5YnpFT01Bd0dBMVVFRVF3Rk1EWXpNREF4Q3pBSkJnTlZCQVlUQWsxWU1RMHdDd1lEVlFRSURBUkRSRTFZTVJNd0VRWURWUVFIREFwRFZVRlZTRlJGVFU5RE1SVXdFd1lEVlFRdEV3eFRRVlE1TnpBM01ERk9Uak14WERCYUJna3Foa2lHOXcwQkNRSVRUWEpsYzNCdmJuTmhZbXhsT2lCQlJFMUpUa2xUVkZKQlEwbFBUaUJEUlU1VVVrRk1JRVJGSUZORlVsWkpRMGxQVXlCVVVrbENWVlJCVWtsUFV5QkJUQ0JEVDA1VVVrbENWVmxGVGxSRk1CNFhEVEkwTURreE9URTNNekl3TlZvWERUSTRNRGt4T1RFM016SXdOVm93Z2R3eEtqQW9CZ05WQkFNVElWSkZSa1ZTUlU1RFNVRlRJRmtnVGtWVVYwOVNTMGxPUnlCVFFTQkVSU0JEVmpFcU1DZ0dBMVVFS1JNaFVrVkdSVkpGVGtOSlFWTWdXU0JPUlZSWFQxSkxTVTVISUZOQklFUkZJRU5XTVNvd0tBWURWUVFLRXlGU1JVWkZVa1ZPUTBsQlV5QlpJRTVGVkZkUFVrdEpUa2NnVTBFZ1JFVWdRMVl4SlRBakJnTlZCQzBUSEVKTlJUQTJNRE15TXpkWE55QXZJRTFGVUUwNE1UQXhNRGswTkRreEhqQWNCZ05WQkFVVEZTQXZJRTFGVUUwNE1UQXhNRGxOUjFSRVJGSXdOREVQTUEwR0ExVUVDeE1HVFVGVVVrbGFNSUlCSWpBTkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQXV4a0prU0VOL20zeWJoWEUySm1GMUVTSkV0N0RWVCt0VGJXS2NOcjV2VmtMQXpyT3pIYWhIajRwMUFLQ3dxazZXM0kyN292bWdER2IxUmtjaEYvTzQ5K0lVTGtNNVRkZ3NJRWd4Nk5vYTNJV0pXaTl0ZmtQYVkrSkYzcG5iQXY0akVjRHkvYTJLK2lTMkZaVVZHQ0taMXM3NEtrWEo1YUZnVi9FNnh0QlBaaFMwQ2o3c1ZaRFQ3d0pxc0pVR09qRkoyRkJkS0FmZGNLTy9qNmlJY2g3RVM0cTd1SlpYdXJFY096ZjhyajRGRVRqZWdnc1lXY2ErVEZabG1IUS8wblE5SGZXWWd6MnFwZFM0TUdrbUdHNlNOdHdoUmtPcklsMEVsay9FZzJrYnhWenFUL2Q0TERhaVI3Ry9kcWlSTEliOVR1UXhhQ291UUM5TVVGTmdDNm5YUUlEQVFBQm94MHdHekFNQmdOVkhSTUJBZjhFQWpBQU1Bc0dBMVVkRHdRRUF3SUd3REFOQmdrcWhraUc5dzBCQVFzRkFBT0NBZ0VBWkFmdTZqanNjWERVSW4zWkFveGV5Z2xGVnMwOVZMb1FpSmNob2xKdFVEaWQxdDB2dlpJb1o1UGkxaDUvbEpkNlNaRXJTK2N3R0lLbXpUT2pERUZrcWQ5OTEvdWJBUzh5WWVzOFZsczJlUWZZcG42ZHZUeThldmU2c2ZFR0F2YnIrb3IvY2tFUHREdklKNXhWYVVDRUo3N0hzWmZLbCtEYm9qTElHRlUvUzRsbWM3bW9pbnpmOGFPUXN1ODJhTDAyemFNNDRiK2tkZUpoYU5HdUtBWUVsTkJUa3U0U2JaNkp5OXRBUlN3ekpKMjBqZk1WU2M5VkxyYndnRFhJNUFCTEU1SVNzbGlyYWcyWUVJa1Y3ejNSdVVqWGViUlN5TVltWmN1VXkzVTRRN1NMSkNXWFkvN0tnQ1FpUnV1bU9jSDFnc3lPZEhJSXZLUUpaQkhIWld5Qy9ibmt6aWtZQjZZZ1hzd0hyc3plVlUvdnRteU45eUVGQ2pMZ2dHOXlwbXhDSTRqa3Fldnc2LytMcnpJaDZzUjdxcjhoMmE1Q1JkYlAydm4xQkIvUWVwOW1rZTRxcmw4ZFJpbUdtM24xNEJYTWI4RlVoT0QrQ1lCdHo0Ym1SVFhTQlpVdlgzb1FycC83SjBDa2hpRDlVZ3lKUEk3V2dUTEtDdmd0L0lxUSs3MGpqa2NDdjNTWEVXdDZLZlYwczVRbEpBcUZ5cEpadi8xcXUyQkxNWWJNL05hR092dnRPckxHejdmajhhdUFGdDFoN3dDWFR0a0NtWnExUDBaZVNSTEprVzV4T093UFhqc2p0WWF0MlNxYXhPVzhBcmxOM2tOcTFJOHBMTndiZ1pKd3FnOGRCcE9ma0gwdG84U1hoSWdzQzdhYXJUMVQ0ZzhSakwyRkJiV2tzVWM9IiBTZWxsbz0icWEzWGdYMzBwTU1kOFJVMmordzlXbTJ2STVhbG1OVCtuTHlZWXpBZU92K0pKUVNuZjdPNGR1RG9XaER5N0pGM1N5b3NhemNZNW42S1AxOFV4TGdZMm91QThZQ09xMGZaSmtFMXpFaXJ4U0xzUEdVaWJKNzdEYUJSQzE3Zjh1ayswb2xEa0o3Q0Rrd0h5cTVmNWJzazdUcHQ3L050ZFNNYklPUkpjOU9Kc0ZEV3JHRmlRaG1DNlFqMXJLUzhsUThnTjZ5TjV0b2ZWNHFDLzBNZzQ1dUN1NFdTTDhha0V3Ly9SbUs1MDhLTmRLK2J0SGc5Z0tKTDdtRlc1K3V1Q2p4TmRLeTc2R0hlMWxaQWtzTEUwbnAycFFHQmZhTENVL2FzbzVEdnpYQU5CaUEzcnk2K2dTNFJwZlNQbjRoRDNvbmkzM1JjZEtJclkvVTdJVjhlc3JhMnhRPT0iPjxjZmRpOkVtaXNvciBSZmM9IkJNRTA2MDMyMzdXNyIgTm9tYnJlPSJSRUZFUkVOQ0lBUyBZIE5FVFdPUktJTkciIFJlZ2ltZW5GaXNjYWw9IjYwMSIgLz48Y2ZkaTpSZWNlcHRvciBSZmM9IlNWQjEwMDUyNzI2MCIgTm9tYnJlPSJTT0NJT1MgRU4gVkVOVEFTIEJBSklPIiBVc29DRkRJPSJHMDMiIFJlZ2ltZW5GaXNjYWxSZWNlcHRvcj0iNjAxIiBEb21pY2lsaW9GaXNjYWxSZWNlcHRvcj0iMzgwNTAiIC8+PGNmZGk6Q29uY2VwdG9zPjxjZmRpOkNvbmNlcHRvIENsYXZlUHJvZFNlcnY9IjgwMTQxOTAyIiBDYW50aWRhZD0iMS4wMDAwMDAiIENsYXZlVW5pZGFkPSJFNDgiIFVuaWRhZD0iVW5pZGFkIGRlIHNlcnZpY2lvIiBEZXNjcmlwY2lvbj0iTWVtYnJlc8OtYSBwb3IgMTIgbWVzZXMiIFZhbG9yVW5pdGFyaW89IjM3MjguNDUiIEltcG9ydGU9IjM3MjguNDUiIE9iamV0b0ltcD0iMDIiIERlc2N1ZW50bz0iMC4wMCI+PGNmZGk6SW1wdWVzdG9zPjxjZmRpOlRyYXNsYWRvcz48Y2ZkaTpUcmFzbGFkbyBCYXNlPSIzNzI4LjQ1IiBJbXB1ZXN0bz0iMDAyIiBUaXBvRmFjdG9yPSJUYXNhIiBUYXNhT0N1b3RhPSIwLjE2MDAwMCIgSW1wb3J0ZT0iNTk2LjU1IiAvPjwvY2ZkaTpUcmFzbGFkb3M+PC9jZmRpOkltcHVlc3Rvcz48L2NmZGk6Q29uY2VwdG8+PC9jZmRpOkNvbmNlcHRvcz48Y2ZkaTpJbXB1ZXN0b3MgVG90YWxJbXB1ZXN0b3NUcmFzbGFkYWRvcz0iNTk2LjU1Ij48Y2ZkaTpUcmFzbGFkb3M+PGNmZGk6VHJhc2xhZG8gQmFzZT0iMzcyOC40NSIgSW1wdWVzdG89IjAwMiIgVGlwb0ZhY3Rvcj0iVGFzYSIgVGFzYU9DdW90YT0iMC4xNjAwMDAiIEltcG9ydGU9IjU5Ni41NSIgLz48L2NmZGk6VHJhc2xhZG9zPjwvY2ZkaTpJbXB1ZXN0b3M+PGNmZGk6Q29tcGxlbWVudG8+PHRmZDpUaW1icmVGaXNjYWxEaWdpdGFsIHhzaTpzY2hlbWFMb2NhdGlvbj0iaHR0cDovL3d3dy5zYXQuZ29iLm14L1RpbWJyZUZpc2NhbERpZ2l0YWwgaHR0cDovL3d3dy5zYXQuZ29iLm14L3NpdGlvX2ludGVybmV0L2NmZC9UaW1icmVGaXNjYWxEaWdpdGFsL1RpbWJyZUZpc2NhbERpZ2l0YWx2MTEueHNkIiBWZXJzaW9uPSIxLjEiIFVVSUQ9IjZmYjgzNDIzLTM4MDQtNDNkNC04OWVmLTIyMzhiYTM1YmJiNCIgRmVjaGFUaW1icmFkbz0iMjAyNS0xMS0xOVQyMjo0Mzo1NyIgUmZjUHJvdkNlcnRpZj0iTFNPMTMwNjE4OVI1IiBTZWxsb0NGRD0icWEzWGdYMzBwTU1kOFJVMmordzlXbTJ2STVhbG1OVCtuTHlZWXpBZU92K0pKUVNuZjdPNGR1RG9XaER5N0pGM1N5b3NhemNZNW42S1AxOFV4TGdZMm91QThZQ09xMGZaSmtFMXpFaXJ4U0xzUEdVaWJKNzdEYUJSQzE3Zjh1ayswb2xEa0o3Q0Rrd0h5cTVmNWJzazdUcHQ3L050ZFNNYklPUkpjOU9Kc0ZEV3JHRmlRaG1DNlFqMXJLUzhsUThnTjZ5TjV0b2ZWNHFDLzBNZzQ1dUN1NFdTTDhha0V3Ly9SbUs1MDhLTmRLK2J0SGc5Z0tKTDdtRlc1K3V1Q2p4TmRLeTc2R0hlMWxaQWtzTEUwbnAycFFHQmZhTENVL2FzbzVEdnpYQU5CaUEzcnk2K2dTNFJwZlNQbjRoRDNvbmkzM1JjZEtJclkvVTdJVjhlc3JhMnhRPT0iIE5vQ2VydGlmaWNhZG9TQVQ9IjAwMDAxMDAwMDAwNzE5NTQ1MzAzIiBTZWxsb1NBVD0iSTlxUW92Vk9Sdnd2cko5eHE4TjYzWXdlV21KUUsrOHlZalBleTlCWnRadVRBUmh0ei91Y0l0ZUhoVWNHUWpRWk9NTDB2VUF3SWhvRHc1eU5abks5R1B4UWp1ZXo0Nm54S0dKNUp3eEQ5cmhwWS9MKytpcHF2aHBFRGY5OUhwZUxMU000NmFadThHLzJhbFd6MEk3M2xlOGdkUHFaWjhyVG9KZmpyM3FoTW95bWJwaTU3amNnWUVPZnlyWDhuajJ5OGVBMityWnRxOUNiQUNHV3Q5TFk2R25KV05nYjlIN3djTGpGcWNpYkNyeERDSnlPaUJYL0prZ1gwS1hjNFhPVkQzZXRrQll2aVRtcmMyWWk0VVVtbDNVUU5TaGV1Rk5TUUNnNkY0bERqSmhGYklqMjJHaG1oSjIyRXZ0c0pLZlcyeFhycDkwYjVxM2NMN1I4WldKQm5RPT0iIHhtbG5zOnRmZD0iaHR0cDovL3d3dy5zYXQuZ29iLm14L1RpbWJyZUZpc2NhbERpZ2l0YWwiIHhtbG5zOnhzaT0iaHR0cDovL3d3dy53My5vcmcvMjAwMS9YTUxTY2hlbWEtaW5zdGFuY2UiIC8+PC9jZmRpOkNvbXBsZW1lbnRvPjwvY2ZkaTpDb21wcm9iYW50ZT4=",
            "xml_nombre_archivo": "CFDI_INV_2025_00005.xml"
        }
    ],
    "response_time": 0.651,
    "timestamp": "2025-12-02T13:49:21.449898"
}
```

## Códigos de Respuesta

### Códigos HTTP Estándar

| Código | Descripción           | Cuándo se usa             |
|--------|-----------------------|---------------------------|
| 200    | OK                    | Petición exitosa          |
| 400    | Bad Request           | Parámetros inválidos      |
| 401    | Unauthorized          | Token inválido o expirado |
| 403    | Forbidden             | Sin permisos de acceso    |
| 404    | Not Found             | Recurso no encontrado     |
| 426    | Upgrade Required      | HTTPS requerido           |
| 500    | Internal Server Error | Error del servidor        |

### Códigos de Error Personalizados

| Código               | Descripción                        |
|----------------------|------------------------------------|
| INVALID_CREDENTIALS  | Credenciales de acceso incorrectas |
| NO_API_ACCESS        | Usuario sin permisos para el API   |
| TOKEN_EXPIRED        | Token de acceso expirado           |
| TOKEN_REVOKED        | Token revocado por el sistema      |
| INVALID_TOKEN        | Token malformado o inválido        |
| INVALID_DATE_FORMAT  | Formato de fecha incorrecto        |
| INVALID_PAYMENT_TYPE | Tipo de pago no válido             |
| INVALID_STATE        | Estado no válido                   |
| HTTPS_REQUIRED       | Se requiere conexión HTTPS         |
| MISSING_PARAMETER    | Falta parámetro requerido          |

---

## Referencia de Campos

### Campos de Factura (55+ campos)

#### Información Básica

| Campo | Tipo    | Descripción         |
|-------|---------|---------------------|
| id    | integer | ID interno en Odoo  |
| name  | string  | Número de factura   |
| folio | string  | Folio de factura    |
| serie | string  | Serie de factura    |
| uuid  | string  | Folio fiscal (UUID) |

#### Fechas

| Campo               | Tipo     | Descripción                |
|---------------------|----------|----------------------------|
| fecha_emision       | datetime | Fecha de emisión           |
| fecha_certificacion | datetime | Fecha de certificación SAT |

#### Emisor

| Campo                 | Tipo   | Descripción           |
|-----------------------|--------|-----------------------|
| emisor.rfc            | string | RFC del emisor        |
| emisor.nombre         | string | Nombre del emisor     |
| emisor.regimen_fiscal | string | Código régimen fiscal |

#### Receptor

| Campo                              | Tipo   | Descripción         |
|------------------------------------|--------|---------------------|
| receptor.rfc                       | string | RFC del receptor    |
| receptor.nombre                    | string | Nombre del receptor |
| receptor.uso_cfdi                  | string | Código uso CFDI     |
| receptor.domicilio_fiscal_receptor | string | Código postal       |

#### Montos

| Campo                              | Tipo    | Descripción                 |
|------------------------------------|---------|-----------------------------|
| montos.subtotal                    | decimal | Subtotal sin impuestos      |
| montos.descuento                   | decimal | Descuento aplicado          |
| montos.total_impuestos_traslados   | decimal | Total impuestos trasladados |
| montos.total_impuestos_retenciones | decimal | Total retenciones           |
| montos.total                       | decimal | Total de la factura         |

#### Conceptos

| Campo                       | Tipo    | Descripción                 |
|-----------------------------|---------|-----------------------------|
| conceptos[].clave_prod_serv | string  | Clave SAT producto/servicio |
| conceptos[].descripcion     | string  | Descripción del concepto    |
| conceptos[].cantidad        | decimal | Cantidad                    |
| conceptos[].valor_unitario  | decimal | Valor unitario              |
| conceptos[].importe         | decimal | Importe total               |

#### Timbre Fiscal

| Campo                                     | Tipo     | Descripción       |
|-------------------------------------------|----------|-------------------|
| complementos.timbre_fiscal.uuid           | string   | UUID del timbre   |
| complementos.timbre_fiscal.fecha_timbrado | datetime | Fecha de timbrado |
| complementos.timbre_fiscal.sello_cfd      | string   | Sello digital CFD |
| complementos.timbre_fiscal.sello_sat      | string   | Sello SAT         |

### Campos de Pago (40+ campos)

#### Información Básica

| Campo | Tipo    | Descripción         |
|-------|---------|---------------------|
| id    | integer | ID interno del pago |
| name  | string  | Número del pago     |
| folio | string  | Folio del REP       |

#### Fechas

| Campo                      | Tipo     | Descripción             |
|----------------------------|----------|-------------------------|
| fechas.date                | date     | Fecha contable          |
| fechas.fecha_pago          | date     | Fecha de pago           |
| fechas.fecha_certificacion | datetime | Fecha certificación SAT |

#### Tipo y Estado

| Campo                      | Tipo   | Descripción                           |
|----------------------------|--------|---------------------------------------|
| tipo_y_estado.payment_type | string | Tipo pago (inbound/outbound/transfer) |
| tipo_y_estado.state        | string | Estado Odoo                           |
| tipo_y_estado.estado_pago  | string | Estado REP en SAT                     |

#### Montos

| Campo                 | Tipo    | Descripción     |
|-----------------------|---------|-----------------|
| montos.amount         | decimal | Monto del pago  |
| montos.amount_signed  | decimal | Monto con signo |
| montos.amount_to_text | string  | Monto en letra  |

#### Partner

| Campo                | Tipo    | Descripción        |
|----------------------|---------|--------------------|
| partner.partner_id   | integer | ID del partner     |
| partner.partner_name | string  | Nombre del partner |
| partner.partner_vat  | string  | RFC del partner    |

#### Información de Pago

| Campo                          | Tipo   | Descripción              |
|--------------------------------|--------|--------------------------|
| informacion_pago.forma_pago_id | string | Código forma de pago SAT |
| informacion_pago.methodo_pago  | string | Método pago (PUE/PPD)    |

#### Datos Bancarios Emisor

| Campo                         | Tipo   | Descripción             |
|-------------------------------|--------|-------------------------|
| banco_emisor.cuenta_emisor    | string | Cuenta bancaria emisora |
| banco_emisor.banco_emisor     | string | Nombre banco emisor     |
| banco_emisor.rfc_banco_emisor | string | RFC banco emisor        |
| banco_emisor.numero_operacion | string | Número de operación     |

#### Datos Bancarios Receptor

| Campo                              | Tipo   | Descripción           |
|------------------------------------|--------|-----------------------|
| banco_receptor.banco_receptor      | string | Nombre banco receptor |
| banco_receptor.cuenta_beneficiario | string | Cuenta beneficiario   |
| banco_receptor.rfc_banco_receptor  | string | RFC banco receptor    |

#### Datos CFDI (REP)

| Campo                            | Tipo   | Descripción                |
|----------------------------------|--------|----------------------------|
| datos_cfdi_rep.folio_fiscal      | string | UUID del REP               |
| datos_cfdi_rep.numero_cetificado | string | Número certificado digital |
| datos_cfdi_rep.selo_digital_cdfi | string | Sello digital CFDI         |
| datos_cfdi_rep.selo_sat          | string | Sello SAT                  |
| datos_cfdi_rep.cadena_origenal   | string | Cadena original            |
| datos_cfdi_rep.codigo_qr         | string | URL código QR              |

#### Facturas Relacionadas

| Campo                                      | Tipo    | Descripción        |
|--------------------------------------------|---------|--------------------|
| facturas_relacionadas[].invoice_id         | integer | ID de la factura   |
| facturas_relacionadas[].uuid               | string  | UUID de la factura |
| facturas_relacionadas[].monto_pago         | decimal | Monto pagado       |
| facturas_relacionadas[].imp_saldo_anterior | decimal | Saldo anterior     |
| facturas_relacionadas[].imp_saldo_insoluto | decimal | Saldo restante     |

#### Archivos Adjuntos

| Campo                        | Tipo    | Descripción                    |
|------------------------------|---------|--------------------------------|
| attachments[].id             | integer | ID del archivo                 |
| attachments[].name           | string  | Nombre del archivo             |
| attachments[].mimetype       | string  | Tipo MIME                      |
| attachments[].content_base64 | string  | Contenido en base64 (solo XML) |

---

## Catálogos SAT

### Uso CFDI (c_UsoCFDI)

| Código | Descripción                                                      |
|--------|------------------------------------------------------------------|
| G01    | Adquisición de mercancías                                        |
| G02    | Devoluciones, descuentos o bonificaciones                        |
| G03    | Gastos en general                                                |
| I01    | Construcciones                                                   |
| I02    | Mobiliario y equipo de oficina por inversiones                   |
| I03    | Equipo de transporte                                             |
| I04    | Equipo de cómputo y accesorios                                   |
| I05    | Dados, troqueles, moldes, matrices y herramental                 |
| I06    | Comunicaciones telefónicas                                       |
| I07    | Comunicaciones satelitales                                       |
| I08    | Otra maquinaria y equipo                                         |
| D01    | Honorarios médicos, dentales y gastos hospitalarios              |
| D02    | Gastos médicos por incapacidad o discapacidad                    |
| D03    | Gastos funerales                                                 |
| D04    | Donativos                                                        |
| D05    | Intereses reales efectivamente pagados por créditos hipotecarios |
| D06    | Aportaciones voluntarias al SAR                                  |
| D07    | Primas por seguros de gastos médicos                             |
| D08    | Gastos de transportación escolar obligatoria                     |
| D09    | Depósitos en cuentas para el ahorro                              |
| D10    | Pagos por servicios educativos                                   |
| S01    | Sin efectos fiscales                                             |
| CP01   | Pagos                                                            |
| CN01   | Nómina                                                           |

### Forma de Pago (c_FormaPago)

| Código | Descripción                         |
|--------|-------------------------------------|
| 01     | Efectivo                            |
| 02     | Cheque nominativo                   |
| 03     | Transferencia electrónica de fondos |
| 04     | Tarjeta de crédito                  |
| 05     | Monedero electrónico                |
| 06     | Dinero electrónico                  |
| 08     | Vales de despensa                   |
| 12     | Dación en pago                      |
| 13     | Pago por subrogación                |
| 14     | Pago por consignación               |
| 15     | Condonación                         |
| 17     | Compensación                        |
| 23     | Novación                            |
| 24     | Confusión                           |
| 25     | Remisión de deuda                   |
| 26     | Prescripción o caducidad            |
| 27     | A satisfacción del acreedor         |
| 28     | Tarjeta de débito                   |
| 29     | Tarjeta de servicios                |
| 30     | Aplicación de anticipos             |
| 31     | Intermediario pagos                 |
| 99     | Por definir                         |

### Método de Pago (c_MetodoPago)

| Código | Descripción                      |
|--------|----------------------------------|
| PUE    | Pago en una sola exhibición      |
| PPD    | Pago en parcialidades o diferido |

### Régimen Fiscal (c_RegimenFiscal)

| Código | Descripción                                                                                |
|--------|--------------------------------------------------------------------------------------------|
| 601    | General de Ley Personas Morales                                                            |
| 603    | Personas Morales con Fines no Lucrativos                                                   |
| 605    | Sueldos y Salarios e Ingresos Asimilados a Salarios                                        |
| 606    | Arrendamiento                                                                              |
| 607    | Régimen de Enajenación o Adquisición de Bienes                                             |
| 608    | Demás ingresos                                                                             |
| 610    | Residentes en el Extranjero sin Establecimiento Permanente en México                       |
| 611    | Ingresos por Dividendos (socios y accionistas)                                             |
| 612    | Personas Físicas con Actividades Empresariales y Profesionales                             |
| 614    | Ingresos por intereses                                                                     |
| 615    | Régimen de los ingresos por obtención de premios                                           |
| 616    | Sin obligaciones fiscales                                                                  |
| 620    | Sociedades Cooperativas de Producción que optan por diferir sus ingresos                   |
| 621    | Incorporación Fiscal                                                                       |
| 622    | Actividades Agrícolas, Ganaderas, Silvícolas y Pesqueras                                   |
| 623    | Opcional para Grupos de Sociedades                                                         |
| 624    | Coordinados                                                                                |
| 625    | Régimen de las Actividades Empresariales con ingresos a través de Plataformas Tecnológicas |
| 626    | Régimen Simplificado de Confianza                                                          |

### Tipo de Comprobante (c_TipoDeComprobante)

| Código | Descripción |
|--------|-------------|
| I      | Ingreso     |
| E      | Egreso      |
| T      | Traslado    |
| N      | Nómina      |
| P      | Pago        |

### Impuestos (c_Impuesto)

| Código | Descripción |
|--------|-------------|
| 001    | ISR         |
| 002    | IVA         |
| 003    | IEPS        |

## FAQ - Preguntas Frecuentes

### ¿Cuánto tiempo permanece válido un token?

Por defecto, los tokens tienen una validez de 1 hora (3600 segundos). Este tiempo se puede configurar al generar el
token usando el parámetro `expires_in`.

### ¿Puedo usar el mismo token para múltiples peticiones?

Sí, puedes reutilizar el mismo token para todas tus peticiones mientras permanezca válido.

### ¿Qué pasa si mi token expira durante el procesamiento?

Debes implementar lógica para detectar errores 401 y renovar el token automáticamente.

### ¿Cuál es el límite máximo de registros por petición?

El límite máximo es 1000 registros por petición. Para conjuntos de datos mayores, usa paginación.

### ¿Los datos se actualizan en tiempo real?

Los datos se actualizan cada vez que se realiza una operación en Odoo. La API consulta directamente la base de datos en
cada petición.

### ¿Puedo filtrar por múltiples criterios simultáneamente?

Sí, puedes combinar múltiples filtros en una sola petición (fecha + RFC + estado, etc.).

### ¿Cómo obtengo el XML completo de una factura?

El XML viene incluido en el campo `archivos.xml_base64` en formato base64. Debes decodificarlo para obtener el
contenido.

### ¿Los pagos incluyen las facturas relacionadas?

Sí, cada pago incluye el array `facturas_relacionadas` con todas las facturas que fueron pagadas.

### ¿Puedo consultar facturas de todas las compañías?

Dependiendo de los permisos del usuario del token, podrás consultar facturas de las compañías a las que tenga acceso.

### ¿Hay límite de peticiones por minuto?

Actualmente no hay límite explícito, pero se recomienda no exceder 60 peticiones por minuto para evitar sobrecarga del
servidor.

---

## Soporte y Contacto

### Reportar Problemas

Si encuentras algún problema con la API:

1. Verifica los logs en Odoo: `BNI API > Logs de API`
2. Revisa esta documentación y la sección de Troubleshooting
3. Contacta al equipo de soporte con:
    - Descripción detallada del problema
    - Endpoint afectado
    - Parámetros utilizados
    - Código de error recibido
    - Logs relevantes

### Contacto

- **Email:** support@extendrix.com
- **Documentación Odoo:** https://www.odoo.com/documentation/16.0/

[//]: # (- **Repositorio:** [Interno BNI])

---


*Documentación generada: 24 de Noviembre, 2025*  
*© 2025 Extendrix eCommerce Services | BNI Guanajuato*  
*Versión de API: 2.2 | Módulo: extendrix_bni_endpoints v16.0*
