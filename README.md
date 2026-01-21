# Documentación AquaCrop para Seasonal

### ¿Qué es AquaCrop?
Herramienta que transforma información atmosférica (clima/tiempo) en información de rendimiento de cultivos.

**AquaCrop** es un modelo de crecimiento de cultivos desarrollado por la **División de Tierras y Aguas de la FAO** para abordar la seguridad alimentaria y evaluar el efecto del entorno y de la gestión agronómica en la producción de cultivos.

Simula la respuesta del rendimiento de los cultivos herbáceos al agua y está especialmente indicado para condiciones en las que el agua constituye un factor limitante clave en la producción agrícola.

AquaCrop equilibra **precisión, simplicidad y robustez**. Para garantizar su amplia aplicabilidad, utiliza solo un pequeño número de parámetros explícitos y variables de entrada mayoritariamente intuitivas, que pueden determinarse mediante métodos sencillos.

### Enlaces

- **GIT (AquaCrop open-source)**: https://github.com/KUL-RSDA/AquaCrop
- **FAO website**: https://www.fao.org/aquacrop/en/
- **Tutorial AquaCrop.jl**: https://gabo-di.github.io/AquaCrop.jl/dev/
- **Tutorial español (YouTube)**: https://www.youtube.com/watch?v=JqvePbXgkNA

---

## 1. Instalar en HPC de ECMWF

### 1.1 Clonar repositorio y chequear

```bash
git clone https://github.com/KUL-RSDA/AquaCrop.git
cd AquaCrop/src
ldd ./aquacrop | egrep "not found" || echo "OK: no missing shared libs"
```

### 1.2 Cargar librerías y compilar



