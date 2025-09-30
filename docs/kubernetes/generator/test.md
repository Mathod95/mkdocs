<form id="configForm">
    <div class="form-group">
        <label for="appname">APPNAME:</label>
        <input type="text" id="appname" name="appname" placeholder="ex. myapp" required>
    </div>

    <div class="form-group">
        <label for="port">APPLICATION_PORT:</label>
        <input type="number" id="port" name="port" placeholder="ex. 443" required>
    </div>

    <div class="form-group">
        <label for="configPath">/CONFIG:</label>
        <input type="text" id="configPath" name="configPath" placeholder="ex. /config/myapp" required>
    </div>

    <div class="form-group">
        <label for="imageTag">DOCKER/IMAGE:TAG:</label>
        <input type="text" id="imageTag" name="imageTag" placeholder="ex. myapp/image:v1" required>
    </div>

    <div class="form-group">
        <label for="enableMonitoring">Activer Monitoring</label>
        <input type="checkbox" id="enableMonitoring" name="enableMonitoring">
    </div>

    <!-- Bouton Generate centré -->
    <button type="button" id="generateButton" class="md-button md-button--primary">Generate</button>
</form>

<h2>Exemple de code avec vos variables :</h2>
<pre><code id="yamlBlock">
services:
  APPNAME:
    restart: unless-stopped
    container_name: APPNAME
    image: DOCKER/IMAGE:TAG
    hostname: APPNAME
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
    networks:
      - saltbox
    labels:
      com.github.saltbox.saltbox_managed: true
      diun.enable: true
      traefik.enable: true
      traefik.http.routers.APPNAME-http.entrypoints: web
      traefik.http.routers.APPNAME-http.middlewares: globalHeaders@file,redirect-to-https@docker,robotHeaders@file,cloudflarewarp@docker,authelia@docker
      traefik.http.routers.APPNAME-http.rule: Host(\`APPNAME.yourdomain.com\`)
      traefik.http.routers.APPNAME-http.service: APPNAME
      traefik.http.routers.APPNAME.entrypoints: websecure
      traefik.http.routers.APPNAME.middlewares: globalHeaders@file,secureHeaders@file,robotHeaders@file,cloudflarewarp@docker,authelia@docker
      traefik.http.routers.APPNAME.rule: Host(\`APPNAME.yourdomain.com\`)
      traefik.http.routers.APPNAME.service: APPNAME
      traefik.http.routers.APPNAME.tls.certresolver: cfdns
    volumes:
      - /opt/APPNAME:/CONFIG
      - /etc/localtime:/etc/localtime:ro

networks:
  saltbox:
    external: true
</code></pre>

<style>
    #configForm .form-group {
        display: flex;
        align-items: center;
        margin-bottom: 15px;  /* Espacer chaque groupe de champs */
    }

    #configForm label {
        margin-right: 10px;  /* Ajouter un espacement entre le label et le champ */
        width: 200px;  /* Définir une largeur pour les labels */
    }

    #configForm input {
        flex: 1;  /* Prendre tout l'espace restant pour l'input */
        padding: 8px;  /* Ajouter un peu de padding pour améliorer l'apparence */
    }

    #generateButton {
        display: block;
        margin: 20px auto;  /* Centrer le bouton */
    }
</style>

<script>
    document.getElementById('generateButton').addEventListener('click', function(event) {
        event.preventDefault();  // Empêcher le rechargement de la page
        
        // Récupérer les valeurs saisies dans le formulaire
        var appname = document.getElementById('appname').value;
        var port = document.getElementById('port').value;
        var configPath = document.getElementById('configPath').value;
        var imageTag = document.getElementById('imageTag').value;
        var enableMonitoring = document.getElementById('enableMonitoring').checked; // Vérifier si la checkbox est cochée

        // Obtenir le bloc YAML
        var yamlBlock = document.getElementById('yamlBlock');
        
        // Remplacer dynamiquement les variables dans le bloc YAML
        var yamlContent = yamlBlock.textContent
            .replace(/APPNAME/g, appname)                   // Remplacer APPNAME par la valeur saisie
            .replace(/APPLICATION_PORT/g, port)            // Remplacer APPLICATION_PORT par la valeur saisie
            .replace(/\/CONFIG/g, configPath)              // Remplacer /CONFIG par la valeur saisie
            .replace(/DOCKER\/IMAGE:TAG/g, imageTag);      // Remplacer DOCKER/IMAGE:TAG par la valeur saisie

        // Vérifier si la case "Activer Monitoring" est cochée
        if (enableMonitoring) {
            console.log('Case Monitoring cochée');

            // Ajouter la ligne de monitoring dans les labels
            var monitoringLine = "      traefik.http.routers.APPNAME.monitoring: APPNAME";

            // Remplacer APPNAME dans la ligne de monitoring avec la valeur du champ `appname`
            monitoringLine = monitoringLine.replace(/APPNAME/g, appname);

            // Vérifier si la ligne de monitoring est déjà présente dans le bloc YAML
            if (!yamlContent.includes(monitoringLine)) {
                // Trouver la section "labels:"
                var labelsIndex = yamlContent.indexOf('labels:');
                if (labelsIndex !== -1) {
                    // Trouver la fin de la section labels (première ligne vide après labels)
                    var endOfLabelsIndex = yamlContent.indexOf('\n', labelsIndex + 7); // Trouver la fin de "labels:"
                    // Ajouter la ligne de monitoring après la section labels
                    yamlContent = yamlContent.slice(0, endOfLabelsIndex) + '\n' + monitoringLine + yamlContent.slice(endOfLabelsIndex);
                }
            }
        } else {
            console.log('Case Monitoring décochée');

            // Retirer la ligne de monitoring du bloc YAML
            var monitoringLine = "      traefik.http.routers.APPNAME.monitoring: APPNAME";
            monitoringLine = monitoringLine.replace(/APPNAME/g, appname);
            
            // Vérifier si la ligne de monitoring est présente et la retirer
            if (yamlContent.includes(monitoringLine)) {
                yamlContent = yamlContent.replace(monitoringLine + '\n', '');
            }
        }

        // Afficher le contenu du YAML dans le bloc code
        yamlBlock.textContent = yamlContent;

        console.log('Contenu après modification:', yamlContent);
    });
</script>
