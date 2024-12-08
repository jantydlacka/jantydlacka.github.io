---
layout: post
title:  "PhpStorm - Typy složek"
---

# PhpStorm - Typy složek

V PhpStormu můžete nastavit různé typy složek, které vám pomohou organizovat váš projekt. Zde jsou některé z možností:

### **Sources Root**
Označuje hlavní zdrojový kód projektu. PhpStorm bude v těchto složkách hledat třídy a další zdrojové soubory. 

V PhpStorm je složka Sources root adresář, který obsahuje hlavní zdrojové soubory projektu. Tato složka je označena jako hlavní místo, kde se nachází zdrojový kód, a PhpStorm ji používá k usnadnění navigace, automatického doplňování kódu a dalších funkcí IDE. Tento adresář je také specifikován v autoload sekci souboru `composer.json`, což umožňuje automatické načítání tříd.

V souboru `composer.json` je source root definován takto:

```json
"autoload": {
	"psr-4": {
    	"App\\": "app"
	}
}
```

### **Test Sources Root**
Označuje složky, které obsahují testovací kód. PhpStorm bude tyto složky používat pro testovací běhy a analýzu kódu.

### **Resource Root**
Označuje složky, které obsahují zdroje, jako jsou obrázky, konfigurace a další soubory, které nejsou zdrojovým kódem.

### **Excluded**
Označuje složky, které mají být vyloučeny z indexování a analýzy kódu. PhpStorm nebude v těchto složkách hledat třídy ani jiné zdrojové soubory.
