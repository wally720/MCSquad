# Parches locales sobre HeliosLauncher

Cambios propios de MCSquad que viven en ficheros que el upstream
(`dscalzi/HeliosLauncher`) también toca. **Hay que comprobarlos en cada merge o rebase
contra upstream**: si se pierden, el síntoma aparece muy lejos de la causa.

## `app/assets/js/processbuilder.js` — guarda de `classpath` en el módulo raíz

En `_resolveServerLibraries()`, el módulo raíz de un servidor entraba al classpath sin
mirar su bandera `classpath`; el launcher solo la respetaba en los submódulos, dentro de
`_resolveModuleLibraries()`. Con NeoForge el módulo raíz es el jar universal, que al
entrar al classpath se duplica contra el module path y hace que el juego crashee al
arrancar.

El parche añade la misma guarda que ya usan los submódulos:

```js
if(mdl.rawModule.classpath ?? true){
    libs[mdl.getVersionlessMavenIdentifier()] = mdl.getPath()
}
```

- **Síntoma si se pierde:** crash al arrancar Minecraft en servidores de NeoForge
  (BootstrapLauncher falla por el universal duplicado).
- **Origen:** commit `baf2a27` de `BelgianDev/Crafted-Launcher-Legacy` ("Don't include
  modloaders that have the classpath field on false"). Ese commit toca también
  `distromanager.js`, pero solo para cambiar la URL de un CDN; eso no forma parte del
  parche.
- No hace falta nada más: ni código específico de NeoForge, ni actualizar `helios-core`
  (vale `~2.3.0`).
