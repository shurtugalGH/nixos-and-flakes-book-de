# Grundlagen der Nix-Sprache

Die Nix-Sprache ist ist essentiell um Konfigurationen zu deklarieren, die von 
Nix erstellt werden sollen. Um ind den Genuss aller Freuden von NixOS und Flakes 
zu kommen, ist es nötig die Grundlagen dieser Sprache zu verstehen.

Die Nix-Sprache ist eine funktionale Sprache. Falls du schon etwas 
Programmiererfahrung mitbringst, solltest du in weniger als zwei Stunden die 
Grundlagen verstehen.

In der Community gibt es bereits einige gute Tutorials zur Nix-Sprache, also 
werde ich hier das Rad nicht neu erfinden. Um anzufangen empfehle ich die 
folgenden Quellen für einen Einblick in die Nix-Sprache:

1. [**Nix Language Basics - nix.dev**](https://nix.dev/tutorials/first-steps/nix-language):  
   Dieses Tutorial bietet einen umfassenden Überblick der Grundlagen der 
   Nix-Sprache und ist für Anfänger empfohlen.
1. [**A tour of Nix**](https://nixcloud.io/tour/?id=introduction/nix):  
   Ein interaktives Onlinetutorial mit Fokus auf die Konstrukte der 
   Programmiersprache und wie Nix algorithmisch zur Problemlösung eingesetzt 
   werden kann.
1. [**Nix Language - Nix Reference Manual**](https://nixos.org/manual/nix/stable/language/):  
   Die offizielle Dukumentation der Nix-Sprache.  
   - nix.dev und andere benutzerfreundliche Tutorials sind nur für den Einstieg 
   geeignet, und **keines von ihnen führt vollständig in die Syntax von Nix 
   ein**. Wenn du also auf eine neue Syntax stößt, die du bisher noch nicht 
   kanntest, schau bitte in dem offiziellen Dokument nach.
1. [**Noogλe**](https://noogle.dev):  
   Noogle ist eine Funktionsbibliotheksuchmaschine ^^. Hier kann man relativ 
   schnell Funktionen und deren Anwendung finden, die man braucht, was sehr 
   praktisch sein kann.

Es ist okay, wenn du dir erstmal einen groben Überblick über die Syntax 
verschaffst. Du kannst jederzeit zurückkommen und dir die Syntax anschauen, 
falls du später etwas findest, was du nicht verstehst.