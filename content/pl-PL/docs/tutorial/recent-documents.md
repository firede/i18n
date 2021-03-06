# Ostatnie dokumenty (Windows & macOS)

Windows i macOS umożliwiają dostęp do listy ostatnich dokumentów, którą można z otworzyć za pomocą JumpList lub dock menu, odpowiednio od używanego systemu.

**JumpList:**

![JumpList ostatnie pliki](https://cloud.githubusercontent.com/assets/2289/23446924/11a27b98-fdfc-11e6-8485-cc3b1e86b80a.png)

**Dock menu:**

![macOS Dock Menu](https://cloud.githubusercontent.com/assets/639601/5069610/2aa80758-6e97-11e4-8cfb-c1a414a10774.png)

Aby dodać plik do listy ostatnich dokumentów możesz użyć następującego polecenia [app.addRecentDocument](../api/app.md#appaddrecentdocumentpath-macos-windows) znajdującego się w API:

```javascript
const { app } = require('electron')
app.addRecentDocument('/Users/USERNAME/Desktop/work.type')
```

Teraz możesz użyć [app.clearRecentDocuments](../api/app.md#appclearrecentdocuments-macos-windows) aby wyczyścić listę:

```javascript
const { app } = require('electron')
app.clearRecentDocuments()
```

## Windows Notatki

Aby móc skorzystać z tej funkcji w systemie Windows, twoja aplikacja musie być zarejestrowana jako moduł obsługi plików typu dokument, w przeciwnym wypadku plik nie pojawi się w JumpList nawet po ich dodaniu. Możesz znaleźć wszystko, po zarejestrowaniu swojej aplikacji w [Rejestracja Aplikacji](https://msdn.microsoft.com/en-us/library/cc144104(VS.85).aspx).

Kiedy użytkownik kliknie na plik z JumpList, nowa instancja twojej aplikacji wystartuje ze ścieżki pliku dodajnej jako argument w wierszu poleceń.

## macOS notatki

Gdy żądany jest plik z listy ostatnich dokumentów (dock menu), to emitowany jest event `open-file` z modułu `app`.