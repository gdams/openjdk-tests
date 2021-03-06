# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

#
# Makefile to run various JVM tests
#

$(info Running make $(MAKE_VERSION))

.PHONY: test clean compile help runtest rmResultFile resultsSummary

.DEFAULT_GOAL := test

help:
	@echo "This makefile is used to build and execute JVM tests. You should specify the following"
	@echo "variables before using this script:"
	@echo "JAVA_BIN  Path to java bin dir which will be used to built and executed JVM tests"
	@echo "SPEC      Should match the current platform (e.g. SPEC=linux_x86-64, SPEC=linux_x86, SPEC=linux_x86-64_cmprssptrs)"

CD        = cd
ECHO      = echo
MKDIR     = mkdir
PWD       = pwd

ifndef JAVA_BIN
$(error Please provide JAVA_BIN value.)
else
export JAVA_BIN:=$(JAVA_BIN)
$(info set JAVA_BIN to $(JAVA_BIN))
endif

ifndef SPEC
$(error Please provide SPEC that matches the current platform (e.g. SPEC=linux_x86-64))
else
export SPEC:=$(SPEC)
$(info set SPEC to $(SPEC))
endif

ifndef JAVA_VERSION
export JAVA_VERSION:=SE90
else
export JAVA_VERSION:=$(JAVA_VERSION)
endif
$(info set JAVA_VERSION to $(JAVA_VERSION))

Q="
D=/

ifneq (,$(findstring win,$(SPEC)))
P=;
else
P=:
endif

ifndef SRC_ROOT
SRC_ROOT := $(shell $(PWD))$(D)..
SRC_ROOT := $(subst \,/,$(SRC_ROOT))
endif
$(info set SRC_ROOT to $(SRC_ROOT))

ifndef BUILD_ROOT
BUILD_ROOT := $(SRC_ROOT)$(D)..$(D)jvmtest
BUILD_ROOT := $(subst \,/,$(BUILD_ROOT))
endif
$(info set BUILD_ROOT to $(BUILD_ROOT))

# set default values
JVM_TEST_ROOT := $(SRC_ROOT)
TEST_GROUP=level.*

# removing " 
JAVA_BIN_TMP := $(subst ",,$(JAVA_BIN))
JDK_HOME := $(JAVA_BIN_TMP)$(D)..
ifeq ($(JAVA_VERSION),SE80)
JDK_HOME := $(JAVA_BIN_TMP)$(D)..$(D)..
endif

ifdef JTREG_DIR
# removing "
JTREG_DIR := $(subst ",,$(JTREG_DIR))
endif

ifdef JTREG_TEST_DIR
# removing "
JTREG_TEST_DIR := $(subst ",,$(JTREG_TEST_DIR))
endif

#######################################
# Set OS, ARCH and BITS based on SPEC
#######################################
WORD_LIST:= $(subst _, ,$(SPEC))
OS:=os.$(word 1,$(WORD_LIST))
ARCH_INFO:=$(word 2,$(WORD_LIST))

BITS=bits.32
ARCH=arch.$(ARCH_INFO)
ifneq (,$(findstring -64,$(ARCH_INFO)))
	BITS=bits.64
	ARCH:=arch.$(subst -64,,$(ARCH_INFO))
else
	ifneq (,$(findstring 390,$(ARCH_INFO)))
		BITS=bits.31
	endif
endif

ifneq ($(DEBUG),)
$(info OS is set to $(OS), ARCH is set to $(ARCH) and BITS is set to $(BITS))
endif

DEFAULT_EXCLUDE=d.*.$(SPEC),d.*.$(ARCH),d.*.$(OS),d.*.$(BITS),d.*.generic-all
ifneq ($(DEBUG),)
$(info DEFAULT_EXCLUDE is set to $(DEFAULT_EXCLUDE))
endif

JAVA_COMMAND:=$(Q)$(JAVA_BIN_TMP)$(D)java$(Q)

#######################################
# Setting AIX specific Macro ADD_JVM_LIB_DIR_ON_AIX, AIX need LIBPATH containing VM directory
#######################################
ifneq (,$(findstring aix,$(SPEC)))
	JAVA_LIB_DIR:=$(JAVA_BIN)$(D)..$(D)lib
	ifneq (,$(findstring cmprssptrs,$(SPEC)))
		ADD_JVM_LIB_DIR_ON_AIX=export LIBPATH=$(Q)$(JAVA_LIB_DIR)$(D)ppc64$(D)compressedrefs$(P)$(LIBPATH)$(Q);
	else
		ifneq (,$(findstring 64,$(SPEC)))
			ADD_JVM_LIB_DIR_ON_AIX=export LIBPATH=$(Q)$(JAVA_LIB_DIR)$(D)ppc64$(D)default$(P)$(LIBPATH)$(Q);
		else
			ADD_JVM_LIB_DIR_ON_AIX=export LIBPATH=$(Q)$(JAVA_LIB_DIR)$(D)ppc$(D)default$(P)$(LIBPATH)$(Q);
		endif
	endif
endif

ifneq ($(DEBUG),)
$(info ADD_JVM_LIB_DIR_ON_AIX is set to $(ADD_JVM_LIB_DIR_ON_AIX))
endif

#######################################
# common dir and jars
#######################################
LIB_DIR=$(JVM_TEST_ROOT)$(D)TestConfig$(D)lib
TESTNG=$(LIB_DIR)$(D)TestNG$(D)testng.jar$(P)$(LIB_DIR)$(D)TestNG$(D)jcommander.jar
JUNIT=$(LIB_DIR)$(D)junit-4.10.jar
RESOURCES_DIR=$(JVM_TEST_ROOT)$(D)TestConfig$(D)resources

#######################################
# cmdlinetester jars
#######################################
CMDLINETESTER_JAR   =$(Q)$(JVM_TEST_ROOT)$(D)cmdline_options_tester$(D)$(JAVA_VERSION)$(D)cmdlinetester.jar$(Q)
CMDLINETESTER_RESJAR=$(Q)$(JVM_TEST_ROOT)$(D)cmdline_options_testresources$(D)$(JAVA_VERSION)$(D)cmdlinetestresources.jar$(Q)

#######################################
# testng report dir
#######################################
TIMESTAMP := $(shell perl $(JVM_TEST_ROOT)$(D)TestConfig$(D)scripts$(D)testKitGen$(D)resultsSummary$(D)getTimeStamp.pl -v)
TESTOUTPUT = $(shell $(PWD))$(D)test_output_$(TIMESTAMP)
REPORTDIR = $(Q)$(TESTOUTPUT)$(D)$@$(Q)

#######################################
# TEST_STATUS
#######################################
TEST_STATUS=if [ $$? -eq 0 ] ; then $(ECHO) $(Q)$@$(Q)$(Q)_PASSED$(Q); else $(ECHO) $(Q)$@$(Q)$(Q)_FAILED$(Q); fi | tee -a TestTargetResult
ifneq ($(DEBUG),)
$(info TEST_STATUS is $(TEST_STATUS))
endif
TEST_SKIP_STATUS=@$(ECHO) $@_SKIPPED | tee -a TestTargetResult

#######################################
# include sub makefile
#######################################
-include $(JVM_TEST_ROOT)$(D)TestConfig$(D)extraSettings
include $(JVM_TEST_ROOT)$(D)TestConfig$(D)specToPlat.mk
include $(JVM_TEST_ROOT)$(D)TestConfig$(D)jvmTest.mk
include $(JVM_TEST_ROOT)$(D)autoGenTest.mk

#######################################
# generic target
#######################################
_TESTTARGET = $(firstword $(MAKECMDGOALS))
TESTTARGET = $(patsubst _%,%,$(_TESTTARGET))
ifneq ($(DEBUG),)
$(info _TESTTARGET is $(_TESTTARGET))
$(info TESTTARGET is $(TESTTARGET))
endif

.PHONY: _$(TESTTARGET) $(TESTTARGET)
$(TESTTARGET): JVM_TEST_ROOT := $(BUILD_ROOT)
_$(TESTTARGET): setup_$(TESTTARGET) $(TESTTARGET)
	@$(ECHO) $@ done

setup_%:
	@$(ECHO)
	@$(ECHO) Running test _$(TESTTARGET)
	$(JAVA_COMMAND) -version

#######################################
# test_dir target
#######################################
TEST_DIRTARGET = $(firstword $(MAKECMDGOALS))
DIRTARGET = $(patsubst test_%,%,$(_TESTTARGET))
ifneq ($(DEBUG),)
$(info _TESTTARGET is $(_TESTTARGET))
$(info TESTTARGET is $(TESTTARGET))
endif

.PHONY: test_$(DIRTARGET)
$(DIRTARGET): JVM_TEST_ROOT := $(BUILD_ROOT)
test_$(DIRTARGET): rmResultFile $(DIRTARGET)_$(JAVA_VERSION) resultsSummary

test: compile runtest
	@$(ECHO) "All Tests Completed"

ifndef JCL_VERSION
export JCL_VERSION:=latest
else
export JCL_VERSION:=$(JCL_VERSION)
endif
$(info set JAVA_VERSION to $(JAVA_VERSION))
# compile all tests under $(SRC_ROOT)
compile:
	ant -f $(SRC_ROOT)$(D)TestConfig$(D)scripts$(D)build_test.xml -DSRC_ROOT=$(SRC_ROOT) -DBUILD_ROOT=$(BUILD_ROOT) -DJAVA_BIN=$(JAVA_BIN) -DJAVA_VERSION=$(JAVA_VERSION) -DJCL_VERSION=$(JCL_VERSION)

#######################################
# failed target
#######################################
FAILEDTAGETS = $(shell $(PWD))$(D)failedtargets.mk

-include $(FAILEDTAGETS)

#######################################
# result Summary
#######################################
TEMPRESULTFILE=TestTargetResult
TAPRESULTFILE=TestTargetResult.tap

rmResultFile:
	$(RM) $(Q)$(TEMPRESULTFILE)$(Q)

resultsSummary:
	@perl $(JVM_TEST_ROOT)$(D)TestConfig$(D)scripts$(D)testKitGen$(D)resultsSummary$(D)resultsSum.pl --failuremk=$(Q)$(FAILEDTAGETS)$(Q) --resultFile=$(Q)$(TEMPRESULTFILE)$(Q) --tapFile=$(Q)$(TAPRESULTFILE)$(Q)

sanity: JVM_TEST_ROOT := $(BUILD_ROOT)
sanity: rmResultFile sanity_$(JAVA_VERSION) resultsSummary

extended: JVM_TEST_ROOT := $(BUILD_ROOT)
extended: rmResultFile extended_$(JAVA_VERSION) resultsSummary

openjdk: JVM_TEST_ROOT := $(BUILD_ROOT)
openjdk: rmResultFile openjdk_$(JAVA_VERSION) resultsSummary

runtest: rmResultFile sanity_$(JAVA_VERSION) extended_$(JAVA_VERSION) openjdk_$(JAVA_VERSION) resultsSummary
.NOTPARALLEL: extended sanity openjdk runtest rmResultFile resultsSummary

# cleanup
clean:
	$(RM) -r $(BUILD_ROOT)
	$(RM) -r $(SRC_ROOT)$(D)TestConfig$(D)test_output_*
	$(RM) $(TEMPRESULTFILE)
	$(RM) $(TAPRESULTFILE)
	$(RM) $(FAILEDTAGETS)