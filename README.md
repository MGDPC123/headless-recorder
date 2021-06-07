# DPC Recorder
1.  Aplikacja startuje z pliku src/popup/index.js. Tam następuje wstrzyknięcie
    głównego komponentu Vue.
2.  Główny komponent znajduej się w src/popup/components/App.vue
3.  Kod zwiazany z nagrywaniem znajduje się w src/content-script/EventRecorder.js - metoda \_recordEvent
4.  W src/code-generators można zerknąć do listy różnych zdarzeń, które moglibyśmy podsłuchiwać w dom-events-to-record.js



## Jak uruchomić aplikację?


1.  Zainstaluj Node.js
2.  W folderze z projektem uruchamiamy komendę "npm i", a później "npm run-script build"
3.  Powinien pojawić się folder "build" z końcową wersją pluginu. Wystarczy wejść do ustawień przeglądarki Chrome -> Zarządzaj rozszerzeniami -> Załaduj rozpakowane i wskazujemy na folder build.
4.  Plugin powinien być już widoczne w przeglądarce



## Credits & disclaimer
The following code is based on Headless recorder (https://github.com/checkly/headless-recorder)

Headless recorder is the spiritual successor & love child of segment.io's
[Daydream](https://github.com/segmentio/daydream) and [ui recorder](https://github.com/yguan/ui-recorder).
Headless Recorder was previously named "Puppeteer Recorder".
