# docker-keycloak-elk

Grâce aux conteneurs instanciés par ce projet docker-compose, nous avons la stack ELK et une instance de Keycloak pour sécuriser :
- Une application Angular
- Une API Springboot qui lit des données dans la base de données du conteneur mysql

## Configuration de Logstash
```
input {

	tcp {
		port => 5000
		id => "socket-server"
		type => "socket-server"
		codec => json
  	}

	http {
		response_headers => {
		"Access-Control-Allow-Origin" => "*"
		"Access-Control-Allow-Headers" => "Content-Type, Access-Control-Allow-Headers, Authorization, X-Requested-With"
		"Access-Control-Allow-Methods" => "*"
		"Access-Control-Allow-Credentials" => "*"
		}
		id => "angular"
		type => "angular"
		port => 5001
		codec => json
	}
}

output {
	if [type] == "socket-server" {
		elasticsearch {
			hosts => "elasticsearch:9200"
			user => "elastic"
			password => "${LOGSTASH_INTERNAL_PASSWORD}"
			#data_stream => "true"
			index => "socket-server-%{+YYYY.MM.dd}"
		}
	}
	if [type] == "angular" {
		elasticsearch {
			hosts => "elasticsearch:9200"
			user => "elastic"
			password => "${LOGSTASH_INTERNAL_PASSWORD}"
			#data_stream => "true"
			index => "angular-%{+YYYY.MM.dd}"
		}
	}
	
}
```

L'API envoie des logs via socket TCP sur le port 5000 du conteneur
L'application Angular envoie des logs via des requêtes HTTP POST sur le port 5001 du conteneur
Grâce au champ type, on peut créer deux index différents dans Elasticsearch puis Kibana

## Visualisation des logs sur Kibana
### API
<img width="1775" alt="image" src="https://user-images.githubusercontent.com/56508650/163175394-f6f74ffb-6ebe-41cc-ab82-313e413651d9.png">

### Front-end
<img width="1775" alt="image" src="https://user-images.githubusercontent.com/56508650/163175805-2661a752-2e99-4c39-a8ed-bed699fe2a6a.png">

## Configuration de Keycloak

Afin de sécuriser le back-end et le front-end, nous utilisons Keycloak.
