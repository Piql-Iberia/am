.DEFAULT_GOAL := help

# Paths for Docker named volumes
AM_PIPELINE_DATA ?= $(HOME)/.am/am-pipeline-data
SS_LOCATION_DATA ?= $(HOME)/.am/ss-location-data


define compose_amauat
	docker-compose -f docker-compose.yml -f docker-compose.acceptance-tests.yml $(1)
endef


create-volumes:  ## Create external data volumes.
	mkdir -p ${AM_PIPELINE_DATA}
	docker volume create \
		--opt type=none \
		--opt o=bind \
		--opt device=$(AM_PIPELINE_DATA) \
			am-pipeline-data
	mkdir -p ${SS_LOCATION_DATA}
	docker volume create \
		--opt type=none \
		--opt o=bind \
		--opt device=$(SS_LOCATION_DATA) \
			ss-location-data

bootstrap: bootstrap-storage-service 

bootstrap-storage-service:  ## Boostrap Storage Service (new database).
	docker-compose exec mysql mysql -hlocalhost -uroot -p12345 -e "\
		DROP DATABASE IF EXISTS SS; \
		CREATE DATABASE SS; \
		GRANT ALL ON SS.* TO 'archivematica'@'%' IDENTIFIED BY 'demo';"
	docker-compose run \
		--rm \
		--entrypoint /src/storage_service/manage.py \
			archivematica-storage-service \
				migrate --noinput
	docker-compose run \
		--rm \
		--entrypoint /src/storage_service/manage.py \
			archivematica-storage-service \
				create_user \
					--username="test" \
					--password="test" \
					--email="test@test.com" \
					--api-key="test" \
					--superuser
	# SS needs to be restarted so the local space is created.
	# See #303 (https://git.io/vNKlM) for more details.
	docker-compose restart archivematica-storage-service
