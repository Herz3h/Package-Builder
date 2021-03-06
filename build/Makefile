# Copyright IBM Corporation 2016
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# Makefile

ifndef KITURA_CI_BUILD_SCRIPTS_DIR
KITURA_CI_BUILD_SCRIPTS_DIR = .
endif

ifndef BUILDER_SCRIPTS_DIR
BUILDER_SCRIPTS_DIR = ${KITURA_CI_BUILD_SCRIPTS_DIR}
endif

UNAME = ${shell uname}

all: buildDebug

buildDebug:
	make SWIFT_BUILD_CONFIGURATION="debug" SWIFTC_FLAGS="-Xswiftc -DDEBUG" _build

buildRelease:
	make SWIFT_BUILD_CONFIGURATION="release" _build

build: buildDebug

_build: custombuild
	@echo --- Running build on $(UNAME)
	@echo --- Build scripts directory: ${BUILDER_SCRIPTS_DIR}
	@echo --- Checking swift version
	swift --version
	@echo --- Checking swiftc version
	swiftc --version
	@echo --- Checking git version
	git --version
	@echo --- Checking git revision and branch
	-git rev-parse HEAD
	-git rev-parse --abbrev-ref HEAD
ifeq ($(UNAME), Linux)
	@echo --- Checking Linux release
	-lsb_release -d
endif
	@echo --- Invoking swift build
	swift build -c ${SWIFT_BUILD_CONFIGURATION} $(SWIFTC_FLAGS)

Tests/LinuxMain.swift:
ifeq ($(UNAME), Linux)
	@echo --- Generating $@
	bash ${BUILDER_SCRIPTS_DIR}/generate_linux_main.sh
endif

test: build Tests/LinuxMain.swift
	@echo --- Invoking swift test
	swift test

refetch:
	@echo --- Removing Packages directory
	rm -rf Packages
	@echo --- Fetching dependencies
	swift package fetch

update:
	@echo --- Updating dependencies
	swift package update

clean: customClean
	@echo --- Invoking swift build --clean
	-swift build --clean
	rm -rf .build

generateXcode: build
	swift package generate-xcodeproj

fixXcode: generateXcode customFixXcode
	bash ${BUILDER_SCRIPTS_DIR}/check_xcodeproj_gem.sh
	-ruby ${BUILDER_SCRIPTS_DIR}/fix_xcode_project.rb \
		${shell bash ${BUILDER_SCRIPTS_DIR}/get_latest_xcodeproj.sh} &>/dev/null

openXcode: fixXcode
	open ${shell bash ${BUILDER_SCRIPTS_DIR}/get_latest_xcodeproj.sh}.xcodeproj

custombuild: # do nothing - for the calling Makefile to override
customFixXcode: # do nothing - for the calling Makefile to override
customClean: # do nothing - for the calling Makefile to override

.PHONY: clean build refetch run test custombuild customFixXcode openXcode generateXcode fixXcode
