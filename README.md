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

# 1. Primeros pasos en HPC de ECMWF

## 1.1 Clonar repositorio, cargar librerías y compilar

```bash
git clone https://github.com/KUL-RSDA/AquaCrop.git
module load intel
module load gcc
cd /home/esp9221/PERM/aquacrop/AquaCrop/src
make
ldd ./aquacrop | egrep "not found" || echo "OK: no missing shared libs"
```
Se ha generado un ejecutable ```src/aquacrop``` y una librería ```src/libaquacrop.so```

Comprobamos que se haya generado todo bien
```bash
ls -lh aquacrop libaquacrop.so
file aquacrop libaquacrop.so
ldd ./aquacrop | egrep "not found" || echo "OK: no missing shared libs"

```

## 1.2 Tescase

```bash
cd ~/PERM/aquacrop/AquaCrop/testcase
ln -sf ../src/aquacrop ./aquacrop
chmod +x ./aquacrop

```

### 1.2.1 Inputs Testcase

- Biología y Fenología
- Climatología interpolada
- Forecast

  testcase/
├── aquacrop -> ../src/aquacrop
├── DATA/
├── LIST/
├── OBS/
├── OUTP/
├── OUTP_REF/
├── PARAM/
└── SIMUL/



# 2. AquaCrop-OSPy

AquaCrop-OSPy es un modelo de cultivo-agua en Python de código abierto.
No es una implementación ni una versión oficial del modelo AquaCrop de la FAO. AquaCrop-OSPy reproduce la mayoría de las funcionalidades disponibles en la V7.1, con la excepción de rutinas para el estrés por salinidad y fertilidad, la gestión de malas hierbas, y los cultivos herbáceos perennes.



#### Enlaces

- **AquaCrop Python**: https://github.com/aquacropos/aquacrop
- **AquaCrop-OSPy Python Tutorial**: https://aquacropos.github.io/aquacrop/
- **Notebook**: https://colab.research.google.com/github/aquacropos/aquacrop/blob/master/docs/notebooks/AquaCrop_OSPy_Notebook_1.ipynb
- **Paper**: https://www.sciencedirect.com/science/article/pii/S0378377421002419?via%3Dihub
