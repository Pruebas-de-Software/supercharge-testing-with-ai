# Testing Aumentado con #IA

Estamos viviendo una revolución en el mundo del software, nos encontramos en un punto donde encontramos Prompts en medio de nuestro código, en medio de nuestras pruebas, lo que antes era código ahora es lenguaje natural.

```python
# 🧠→💻  Natural-language testing "inside your code"
prompt = """
Enumera escenarios y ejemplos para probar exhaustivamente la función
calcular_promedio(): casos normales, de borde y entradas inválidas.
"""

response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": prompt}],
)

print(response.choices[0].message.content)  # ← pruebas listas para usar

```


La IA alcancaza una saturación en torno al 96% precisión en preguntas científicas de nivel PhD: la IA está escalando barreras que antes parecían inalcanzables (hace menos de un año era 78%)

![AIPhdBm](https://github.com/Pruebas-de-Software/supercharge-testing-with-ai/blob/main/material/AI_performance_on_a_set_of_Ph.D.-level_science_questions2026.png)

[ref: AI Benchmarking Hub](https://epoch.ai/benchmarks/gpqa-diamond?view=graph&tab=release-date)

El Testing aumentado con IA, puede apalancarse de estos avances para asegurar la calidad de la próxima generación de soluciones digitales. 

Este mundo es alucinante, el mercado es enorme y crece a cada minuto, asegurar la calidad de estos nuevos ecosistemas es fundamental.

La ingeniería de software moderna exige ciclos de entrega cada vez más rápidos y rigurosos. Al integrar **IA generativa** y prácticas de *prompt engineering* con **testing automatizado** y herramientas de última generación, ampliamos la cobertura y reducimos el tiempo de detección de defectos sin sacrificar la calidad.

En este repositorio encontrarás

## Temas

Primera versión del material de **INF331 — Pruebas de Software, UTFSM**, con un caso práctico transversal, prompts, ejemplos y actividades.

[Leer el material completo](material/curso-inf331.md)

- [Prompt engineering](material/curso-inf331.md#prompt-engineering)
  - [Primeros prompts](material/curso-inf331.md#primeros-prompts)
  - [Multiprompts](material/curso-inf331.md#multiprompts)
- [Context Engineering](material/curso-inf331.md#context-engineering)
- [Requerimientos](material/curso-inf331.md#requerimientos)
  - [Requerimientos funcionales](material/curso-inf331.md#requerimientos-funcionales)
  - [Requerimientos no funcionales](material/curso-inf331.md#requerimientos-no-funcionales)
  - [Reglas de negocio](material/curso-inf331.md#reglas-de-negocio)
- [Historias de usuario](material/curso-inf331.md#historias-de-usuario)
- [Criterios de aceptación](material/curso-inf331.md#criterios-de-aceptación)
- [Casos de prueba](material/curso-inf331.md#casos-de-prueba)
- [MCP (Model Context Protocol)](material/curso-inf331.md#mcp--model-context-protocol)
  - [Playwright MCP](material/curso-inf331.md#playwright-mcp)
- [Trabajo integrador](material/curso-inf331.md#trabajo-integrador)

---

# 📝Licencia
MIT © 2025 Greentest.ai.
