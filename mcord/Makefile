# mcord/Makefile

CONFIG_DIR         := $(dir $(abspath $(lastword $(MAKEFILE_LIST))))
COMMON_DIR         := $(abspath ../common )

.DEFAULT_GOAL      := xos

DOCKER_PROJECT     ?= mcord
BOOTSTRAP_PROJECT  ?= mcordbs

xos: prereqs config_dirs xos_download cord_services cord_libraries bootstrap onboarding podconfig

onboarding:
	@echo "[ONBOARDING]"
	# on-board any services here
	bash $(COMMON_DIR)/wait_for_onboarding_ready.sh $(XOS_BOOTSTRAP_PORT) xos
	$(RUN_TOSCA_BOOTSTRAP) $(COMMON_DIR)/tosca/disable-onboarding.yaml
	sudo cp id_rsa key_import/vBBU_rsa
	sudo cp id_rsa.pub key_import/vBBU_rsa.pub
	sudo cp id_rsa key_import/vPGWC_rsa
	sudo cp id_rsa.pub key_import/vPGWC_rsa.pub
	sudo cp id_rsa key_import/onos_rsa
	sudo cp id_rsa key_import/onos_rsa.pub
	$(RUN_TOSCA_BOOTSTRAP) $(LIBRARY_DIR)/ng-xos-lib/ng-xos-lib-onboard.yaml
	$(RUN_TOSCA_BOOTSTRAP) $(SERVICE_DIR)/openstack/xos/openstack-onboard.yaml
	$(RUN_TOSCA_BOOTSTRAP) $(SERVICE_DIR)/vBBU/xos/vBBU-onboard.yaml
	$(RUN_TOSCA_BOOTSTRAP) $(SERVICE_DIR)/vPGWC/xos/vPGWC-onboard.yaml
	$(RUN_TOSCA_BOOTSTRAP) $(SERVICE_DIR)/vtn/xos/vtn-onboard.yaml
	$(RUN_TOSCA_BOOTSTRAP) $(SERVICE_DIR)/onos-service/xos/onos-onboard.yaml
	# $(RUN_TOSCA_BOOTSTRAP) $(SERVICE_DIR)/fabric/xos/\fabric-onboard.yaml
	$(RUN_TOSCA_BOOTSTRAP) synchronizers.yaml
	$(RUN_TOSCA_BOOTSTRAP) $(COMMON_DIR)/tosca/enable-onboarding.yaml
	bash $(COMMON_DIR)/wait_for_onboarding_ready.sh $(XOS_BOOTSTRAP_PORT) services/ng-xos-lib
	bash $(COMMON_DIR)/wait_for_onboarding_ready.sh $(XOS_BOOTSTRAP_PORT) services/vbbu
	bash $(COMMON_DIR)/wait_for_onboarding_ready.sh $(XOS_BOOTSTRAP_PORT) services/vpgwc
	bash $(COMMON_DIR)/wait_for_onboarding_ready.sh $(XOS_BOOTSTRAP_PORT) services/vtn
	bash $(COMMON_DIR)/wait_for_onboarding_ready.sh $(XOS_BOOTSTRAP_PORT) services/onos
	# bash $(COMMON_DIR)/wait_for_onboarding_ready.sh $(XOS_BOOTSTRAP_PORT) services/fabric
	bash $(COMMON_DIR)/wait_for_onboarding_ready.sh $(XOS_BOOTSTRAP_PORT) xos
	bash $(COMMON_DIR)/wait_for_xos_port.sh $(XOS_UI_PORT)

podconfig: nodes.yaml images.yaml
	@echo "[PODCONFIG]"
	$(RUN_TOSCA) setup.yaml
	$(RUN_TOSCA) nodes.yaml
	$(RUN_TOSCA) images.yaml

vtn: vtn-external.yaml
	$(RUN_TOSCA) vtn-external.yaml

vtn-external.yaml:
	export SETUPDIR=$(CONFIG_DIR); bash ./make-vtn-external-yaml.sh

mcord:
	$(RUN_TOSCA) mgmt-net.yaml
	$(RUN_TOSCA) mcord.yaml

ceilometer_custom_images: images/ceilometer-service-trusty-server-multi-nic.img images/ceilometer-trusty-server-multi-nic.img
	#TODO: Add logic for service VM custom image (ceilometer-service-trusty-server-multi-nic)
	bash -c "source ./admin-openrc.sh; glance image-show ceilometer-trusty-server-multi-nic || glance image-create --name ceilometer-trusty-server-multi-nic --disk-format qcow2 --file ./images/ceilometer-trusty-server-multi-nic.compressed.qcow2 --container-format bare"

#vsg_custom_images: images/vsg-1.1.img
#	bash -c "source ./admin-openrc.sh; glance image-show vsg-1.1 || glance image-create --name vsg-1.1 --disk-format qcow2 --file ./images/vsg-1.1.img --container-format bare"

images/ceilometer-trusty-server-multi-nic.img: images
	bash -c "source ./admin-openrc.sh; glance image-show ceilometer-trusty-server-multi-nic || wget http://www.vicci.org/cord/ceilometer-trusty-server-multi-nic.compressed.qcow2 -P ./images"

images/ceilometer-service-trusty-server-multi-nic.img: images
	#TODO: Add logic for service VM custom image (ceilometer-service-trusty-server-multi-nic) once image is uploaded into vicci portal
	#bash -c "source ../setup/admin-openrc.sh; glance image-show ceilometer-service-trusty-server-multi-nic || wget http://www.vicci.org/cord/ceilometer-service-trusty-server-multi-nic.compressed.qcow2 -P ./images"

#images/vsg-1.1.img: images
#	[ -s images/vsg-1.1.img ] || wget http://www.vicci.org/cord/vsg-1.1.img -P ./images

onboard-slicing: slicing-ui
	@echo "[ONBOARDING]"
	# on-board slicing services here
	bash $(COMMON_DIR)/wait_for_onboarding_ready.sh $(XOS_BOOTSTRAP_PORT) xos
	$(RUN_TOSCA_BOOTSTRAP) $(COMMON_DIR)/disable-onboarding.yaml
	$(RUN_TOSCA_BOOTSTRAP) $(SERVICE_DIR)/mcord_slicing_ui/xos/mcord-slicing-onboard.yaml
	bash $(COMMON_DIR)/wait_for_onboarding_ready.sh $(XOS_BOOTSTRAP_PORT) services/mcord_slicing_ui
	bash $(COMMON_DIR)/wait_for_onboarding_ready.sh $(XOS_BOOTSTRAP_PORT) xos
	bash $(COMMON_DIR)/wait_for_xos_port.sh $(XOS_UI_PORT)

slicing-ui:
	$(RUN_TOSCA) ./mcord-slicing-ui.yaml
