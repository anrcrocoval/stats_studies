EXEC = java -jar target/model_deviation-1.0-SNAPSHOT.jar
LIST_OF_N = 10 25 100
LIST_OF_TRANSFORMATIONS = RIGID AFFINE
# LIST_OF_LAYOUTS = uniform gaussian
LIST_OF_LAYOUTS = gaussian

define UNIFORM_SOURCE_DATASET_RULE
sourceDataset_n$(1)_uniform.csv:
	$(EXEC) --width 512 --height 512 -n $(1) generateUniformDataset > $$@
endef

define GAUSSIAN_SOURCE_DATASET_RULE
sourceDataset_n$(1)_gaussian.csv:
	$(EXEC) --width 512 --height 512 -n $(1) generateOutlierDataset --fiducial-configuration-covariance 500 0 0 500 > $$@
endef

define TRANSFORMATION_RULE
$(1).csv:
	$(EXEC) generateTransformation --transformation $(1) > $$@
endef

define RESULT_NAME
result_model_$(1)_transfo_$(2)_n$(3)_layout_$(4).csv
endef

define RESULT_RULE
result_model_$(1)_transfo_$(2)_n$(3)_layout_$(4).csv: $(2).csv sourceDataset_n$(3)_$(4).csv testSourceDataset.csv
	$(EXEC) -a 0.95 --noise-covariance 1 0 0 1 --transformation-model $(1) leaveOneOutSimulation -N 10000 --transformation $(2).csv --source-dataset sourceDataset_n$(3)_$(4).csv --test-source-dataset testSourceDataset.csv > $$@
endef

RESULT_FILES = $(foreach model,$(LIST_OF_TRANSFORMATIONS),$(foreach transformation,$(LIST_OF_TRANSFORMATIONS),$(foreach n,$(LIST_OF_N),$(foreach layout,$(LIST_OF_LAYOUTS),$(call RESULT_NAME,$(model),$(transformation),$(n),$(layout))))))

all: $(RESULT_FILES) layout.png
	Rscript post_process.R

# %.png: %.csv
# 	$(EXEC) --width 1024 --height 1024 generateImageFromCsv -i $< -o $@

layout.png: testSourceDataset.csv sourceDataset_n100_gaussian.csv
	$(EXEC) --width 1024 --height 1024  generateImageFromCsv -i testSourceDataset.csv -i sourceDataset_n100_gaussian.csv -o $@

testSourceDataset.csv:
	$(EXEC) -n 100 --width 1024 --height 1024 generateTestDataset  > $@

# likelihood_ratio_rigid: sourceDataset_n100_gaussian.csv RIGID.csv
# 	$(EXEC) --noise-covariance 100 0 0 100 likelihood --source-dataset sourceDataset_n100_gaussian.csv --transformation RIGID.csv -N 1000 > $@

clean:
	rm -f -- result_* sourceDataset_* likelihood_ratio_* testSourceDataset.csv sourceDataset_* RIGID.csv AFFINE.csv SIMILARITY.csv

reset: clean all

$(foreach model,$(LIST_OF_TRANSFORMATIONS),$(eval $(call TRANSFORMATION_RULE,$(model))))
$(foreach n,$(LIST_OF_N),$(eval $(call UNIFORM_SOURCE_DATASET_RULE,$(n))))
$(foreach n,$(LIST_OF_N),$(eval $(call GAUSSIAN_SOURCE_DATASET_RULE,$(n))))
$(foreach model,$(LIST_OF_TRANSFORMATIONS),$(foreach transformation,$(LIST_OF_TRANSFORMATIONS),$(foreach n,$(LIST_OF_N),$(foreach layout,$(LIST_OF_LAYOUTS),$(eval $(call RESULT_RULE,$(model),$(transformation),$(n),$(layout)))))))
