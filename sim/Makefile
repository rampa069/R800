# Verilator models of the R800 core and its tests (pasmo for the .asm, see README)
RTL  = ../rtl/r800_cpu.v ../rtl/r800_alu.v ../rtl/r800_reg.v
VFLG = --cc --exe --build -O3 -Wno-fatal -Wno-lint -Wno-style --top-module R800
TESTS = multest pfxtest blktest iotest

all: obj_tb/vtb obj_tbw/vtbw obj_tbi/vtbi obj_tbio/vtbio

define model
obj_$(1)/v$(1): $(1).cpp $$(RTL)
	verilator $$(VFLG) -Mdir obj_$(1) $$(RTL) $(1).cpp -o v$(1)
endef
$(foreach m,tb tbw tbi tbio,$(eval $(call model,$(m))))

%.com: %.asm
	pasmo $< $@

cpm.bin: cpm.asm
	pasmo $< $@

# unit tests: expected output in README
test: all cpm.bin $(addsuffix .com,$(TESTS)) inttest.com halttest.com io2test.com
	@for t in $(TESTS); do echo "== $$t"; ./obj_tb/vtb cpm.bin $$t.com 5000000 | tr -d '\r'; done
	@echo "== IOTEST (random waits)"; ./obj_tbw/vtbw cpm.bin iotest.com 400000 | tr -d '\r'
	@echo "== INTTEST"; ./obj_tbi/vtbi cpm.bin inttest.com 20000000 | tr -d '\r'
	@echo "== HALTTEST"; ./obj_tbi/vtbi cpm.bin halttest.com 40000000 | tr -d '\r'
	@echo "== IO2TEST"; ./obj_tbio/vtbio cpm.bin io2test.com 200000 | tr -d '\r'

# zexdoc / zexall (ZEXDOC.COM / ZEXALL.COM, not in this repo): hours
zexdoc: obj_tb/vtb cpm.bin
	./obj_tb/vtb cpm.bin ZEXDOC.COM | tee zexdoc.log
zexall: obj_tb/vtb cpm.bin
	./obj_tb/vtb cpm.bin ZEXALL.COM | tee zexall.log

clean:
	rm -rf obj_* *.com cpm.bin

.PHONY: all test zexdoc zexall clean
