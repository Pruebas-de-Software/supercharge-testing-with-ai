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


La IA alcancaza 78% de precisión en preguntas científicas de nivel PhD: la IA está escalando barreras que antes parecían inalcanzables.

![AIPhdBm](https://github.com/Pruebas-de-Software/supercharge-testing-with-ai/blob/main/material/AI_performance_on_a_set_of_Ph.D.-level_science_questions.png)

[ref: AI Benchmarking Hub](https://epoch.ai/data/ai-benchmarking-dashboard)

El Testing aumentado con IA, puede apalancarse de estos avances para asegurar la calidad de la próxima generación de soluciones digitales. 

Este mundo es alucinante, el mercado es enorme y crece a cada minuto, asegurar la calidad de estos nuevos ecosistemas es fundamental.

La ingeniería de software moderna exige ciclos de entrega cada vez más rápidos y rigurosos. Al integrar **IA generativa** y prácticas de *prompt engineering* con **testing automatizado** y herramientas de última generación, ampliamos la cobertura y reducimos el tiempo de detección de defectos sin sacrificar la calidad.

En este repositorio encontrarás

## Temas
- Prompt engineering
  - Primeros Prompts
  - Multiprompts
- Context Engineering
- Requerimientos
  - Requerimientos Funcionales
  - Requeimientos No funcionales
  - Reglas de Negocio
- Historias de Usuario
- Criterios de Aceptación
- Casos de Pruebas
- MCP (Model Context Protocol)
  - Playwright MCP

---

# 📝Licencia
MIT © 2025 Greentest.ai.
