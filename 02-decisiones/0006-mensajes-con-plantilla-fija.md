# 0006 · Mensajes con plantilla fija

**Estado:** Aceptada · **Fecha:** 2026-11-22

## Decisión
El texto de cada alerta lo arma Python con una plantilla fija (`reportes/mensajes.py`). OpenClaw/Gemini solo lo entrega. Si en el futuro Gemini agrega contexto, va **debajo** del mensaje y nunca cambia cifras.

## Por qué
Un modelo de lenguaje puede redondear, inventar o cambiar una cuota. Una cifra equivocada en una alerta de apuesta cuesta dinero.
