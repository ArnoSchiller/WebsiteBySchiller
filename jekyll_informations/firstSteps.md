Dokumentation: https://jekyllrb.com/docs/home/

1. Erstellen eines neuen Projekts 
    - in entsprechenden Ordner wechseln und cmd öffnen
    - cmd: jekyll new byschiller_blog

2. Webseite starten 
    - cd byschiller_blog
    - bundle exec jekyll serve
        "bundle exec" muss nur beim ersten Mal ausführen dabei sein  
    - Webseite wird gestartet und ist unter http://127.0.0.1:4000/ erreichbar

3. Konfigurationen
    die config.yml enthält alle wichtigen Konfigurationen, die angepasst werden können, z.B.:
        title: Projects by Schiller 
        email: arno-schiller@hotmail.de
        github_username:  ArnoSchiller

4. Einbinden von GitHub Pages
    Dokumentation: https://jekyllrb.com/docs/github-pages/
    Quelle: https://www.youtube.com/watch?v=fqFjuX4VZmU
    Webseite für ein Projekt:
        - Projekt/Repository erstellen (z.B. WebsiteBySchiller) und klonen
            git clone ...
        - config.yml überarbeiten (GitHub Domain):
            baseurl: "WebsiteBySchiller"   # Repo name
        - git bash öffnen und Branch wechseln zu gh-pages 
            git checkout -b gh-pages
        - sicherstellen, dass sich alle erzeugten Dateien im root Ordner liegen und nicht in einem Unterverzeichnis
        - Dateien zu git hinzufügen
            git add .
        - und commiten
            git commit -m "initial commit"
        - Dateien pushen
            git push origin gh-pages
        - Dateien in GitHub überprüfen und Einstellungen anpassen:
            Source: gh-pages branch  
        - für eigene Domain die url in der config anpassen 
            url: "http://byschiller.de"
        - CNAME Datei zum Repo hinzufügen (automatisch, wenn custom domain in den Einstellungen festgelegt wird)

5. Frontmatter
    alle Seiten von jekyll besitzen frontmatter, diese enthalten wichitige Informationen über die Seite, z.B.:
        ---
        layout: post
        title:  "Welcome to Jekyll!"
        date:   2021-01-24 19:50:05 +0100
        categories: jekyll update
        ---
    es können weitere Informationen hinzugefügt werden, dabei sind YAML und JSON als Notation möglich (Key-Value-Paare), z.B.:
        ---
        layout: post
        title:  "Welcome to bySchiller!"
        date:   2021-01-24 19:50:05 +0100
        categories: general update
        author: "Arno Schiller"
        ---