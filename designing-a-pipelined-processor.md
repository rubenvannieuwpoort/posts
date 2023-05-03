# Designing a pipelined processor

In this post I will lay down a VHDL template that can be used for the design of a stage in a pipelined CPU.

A few months ago I set out to design an in-order, [pipelined CPU](https://en.wikipedia.org/wiki/Instruction_pipelining) in VHDL. The goal is to synthesize it on a [Numato Mimas v2 development board](https://numato.com/product/mimas-v2-spartan-6-fpga-development-board-with-ddr-sdram/).

Now, after a few months, the basis of the CPU is there, but there is still a lot of work to do. The pipelining itself was a bigger challenge than I thought it would be.

The idea of a pipelined processor is that instructions flow through the different stages of the processor at the same time. So, if a processor has stages X, Y, and Z, in a given clock cycle it might execute stage Z for instruction 1, stage Y for instruction 2, and stage X for instruction 3. The next clock cycle, the processor might be finished with instruction 1 and start executing instruction 4 (while instructions 2 and 3 are still "in flight"). So the processor will execute stage Z for instruction 2, stage Y for instruction 3, and stage X for instruction 4. (This is a very simple scenario, in practice there can reasons why this neat flow of every instruction moving to the next stage can be interrupted)

This is opposed to a multi-cycle processor,  that was all I needed to know. While that's true, it's not the whole story. The thing that tripped me up is pipeline hazards.

In a pipelined processor, instructions execute in different stages. There can be dependencies between different instructions, in the sense that a certain stage of a certain instruction needs to be executed before a certain stage of another instruction after it. In these cases, the processor might need to delay the execution of the second stage. While pipelined processors are usually able to execute much faster then his complicates the design.

To make this concrete, consider the following assembly program:
```
sub r1, r2
mul r1, r3
```

It subtracts register `r2` from register `r1` and then multiplies it by register `r3`. Of course, we want to result in `r1` to be correct, so before the multiplication can be executed the result of the subtraction needs to be in `r1`.

Now suppose the processor that executes these instructions has a typical 5-stage RISC pipeline:
  1. Instruction fetch
  2. Instruction decode
  3. Execute
  4. Memory
  5. Writeback

This means that before the value of `r1` can be read for the `mul r1, r3` instruction, the "writeback" stage of the `sub r1, r2` needs to be completed. In this 5-stage RISC pipeline, the registers are read in the second stage. This means that at a certain point, the processor needs to wait before executing the instruction decode for the `mul` instruction. In an in-order processor, this means that the instruction decode, execute, and memory stages do no useful work until the writeback of the `sub` instruction is done.

Specifically, this is known as a "read after write" (RAW) hazard. If some stages are doing nothing, we say that there are "pipeline bubbles" or that the pipeline has "stalled".

Read after write hazards not the only reason that the pipeline can be stalled. It is also possible that a memory read takes many cycles and the othere stages need to wait until the read is done.

Anyway, the point I am trying to bring across is that designing a pipelined CPU is a bit more difficult than I anticipated. It's not *that* much more difficult, but especially as a beginner it can really trip you up. I wrote a lot of prototypes, but all of them failed at least one of the constraints I required:
  1. The general approach feels right
  2. The implementation is obviously correct
  3. The implementation is easy enough to understand so that I can make changes later, if necessary

I iterated on this for weeks, but I still felt like there should be a better way, since I could find almost nothing written about the problems I faced. Finally, I found the article ["Strategies for pipelining logic"](https://zipcpu.com/blog/2017/08/14/strategies-for-pipelining.html) from a more experienced hardware engineer who implemented a CPU. It described *exactly* the problems I faced and it comes up with conceptually the same solution as I came up with. This was a huge relief, as it verified my thought process. It also explains the conceptual approach very nicely, which cleared up my head a bit, allowed me to double-down on one approach, and implement that well.

(I still wonder why there are not more articles about this topic -- I am not sure if it is too obscure or if experienced engineers consider it too trivial to write about... Or maybe some combination of both?)


## Iterating through the pipeline logic

With the power of hindsight, it is easy to pretend that I took a straight path from naive initial idea of a CPU pipeline stage to an implementation that handles the pipelines stalls reasonably elegantly. Let's do just that.

So, at the beginning of this project I naively thought of a CPU stage like this:
```
entity stage_template is
	port(
		clk: in std_logic;
		input: in previous_stage_output_type;
		output: out stage_output_type := DEFAULT_STAGE_OUTPUT
	);
end memory;

architecture Behavioral of memory is
begin
	process(clk)
	begin
		if rising_edge(clk) then
			output <= f(input);
		end if;
	end process;
end Behavioral;
```

With such an implementation, the output will be given one clock cycle after the input is received. This is fine for simple pipelines. However, most CPU's will need a pipeline that is able to handle pipeline stalls. One approach is to introduce a `hold_in` signal, which signals to the stage that it should not change the output.

So, we add an `hold_in` input signal and a `hold_out` output signal to the entity, and change the output only if `hold_in` is low. Further, we transfer the value of the `hold_in` signal to the `hold_out` signal on a rising edge of the clock. The `hold_out` signal is meant to be connected to the `hold_in` signal of the stage *before* it. Similarly, the `hold_in` signal is an input from the stage *after* it. A stage raises `hold_out` to signal that it is not ready to receive new data just yet, so the preceding stages should not send data down the pipeline.

```
architecture Behavioral of memory is
begin
	process(clk)
	begin
		if rising_edge(clk) then
			hold_out <= hold_in;
			if hold_in = '0' then
				output <= f(input);
			end if;
		end if;
	end process;
end Behavioral;
```

While the idea behind this implementation is valid, instructions will get lost whenever the `hold` signal gets raised. A stage raises `hold_out` signals that it could not process the data it just received (either because the stage after it is not ready yet, or because of some other reason). But as far as the preceding pipeline stage is concerned, the data is processed and sent to the current stage. So, it is up to the current stage to handle input that is sent but that can't be processed yet.

To solve this, a stage needs to store the instruction that it wasn't able to process yet. This means that when the `hold_in` signal goes high, the current input needs to be buffered -- the next stage will not able to process the output corresponding to this input, and we can't count on the previous stage to hold this input (we can raise `hold_out`, but it will only be effective the next clock cycle).

To buffer the input, we introduce a register `pending_input`. We assume that `current_stage_output_type` has a `valid` signal of type `std_logic`. If this is `0`, the input is not valid (e.g. a pipeline bubble). If `hold_in` is raised and the input is valid, it will get stored in `pending_input`. The input will also need to be selected from either `input` or `pending_input`. For this, we use a variable `v_input`.

```
architecture Behavioral of memory is
	signal pending_input: previous_stage_output_type := DEFAULT_PREVIOUS_STAGE_OUTPUT;
begin
	hold_out <= pending_input.valid;

	process(clk)
		variable v_input: previous_stage_output_type;
		signal pending_input: previous_stage_output_type := DEFAULT_PREVIOUS_STAGE_OUTPUT;
	begin
		if rising_edge(clk) then
			if pending_input.valid = '1' then
				v_input := pending_input;
			else
				v_input := input;
			end if;

			if hold_in = '0' then
				-- set output as a function of v_input (and possibly some state or other inputs)
				output <= f(v_input, state, ...)
				pending_input <= DEFAULT_PREVIOUS_STAGE_OUTPUT;
			else
				pending_input <= v_input;
			end if;
		end if;
	end process;
end Behavioral;
```

Up to now we only considered the case where the `hold_in` was raised by the next stage. However, some stages can also introduce a pipeline stall themselves. For this, we introduce a variable `v_stall`, which is set together with the output based on the input from the previous stage `v_input`, and possibly the state of the stage or other inputs to the stage.

Now, we still only change the output when `hold_in` is low. When `v_stall` is high, the output should be set to `DEFAULT_PREVIOUS_STAGE_OUTPUT` (for simplicity, it is assumed that `f` handles this). We buffer the input when either `hold_in` or `v_stall` is high, since both imply that the current input can not be handled. If they are both low, it means the current input is handled, so the buffered input can be cleared (either `v_input` originated from the buffered input, in which case it can be cleared, or the buffer was already empty, in which case it doesn't matter if it is cleared).

```
architecture Behavioral of memory is
	signal pending_input: previous_stage_output_type := DEFAULT_PREVIOUS_STAGE_OUTPUT;
begin
	hold_out <= pending_input.valid;

	process(clk)
		variable v_input: previous_stage_output_type;
		signal pending_input: previous_stage_output_type := DEFAULT_PREVIOUS_STAGE_OUTPUT;
	begin
		if rising_edge(clk) then
			if pending_input.valid = '1' then
				v_input := pending_input;
			else
				v_input := input;
			end if;

			if hold_in = '0' then
				-- set output and v_stall as a function of v_input (and possibly some state or other inputs)
			end if;

			if hold_in = '0' and v_stall = '0' then
				pending_input <= DEFAULT_PREVIOUS_STAGE_OUTPUT;
			else
				pending_input <= v_input;
			end if;
		end if;
	end process;
end Behavioral;
```

At the cost of having a lot of complexity, we finally have a template that is capable of handling pipeline stalls.

As a bonus, we can consider what we need to add if we need to talk to a component outside the pipeline that expects *commands*. These commands should be transient, that is, the signals should be asserted for a single cycle only. As such, these signals should not be held high when the `hold_in` signal is high. This can be done simply by setting the transient output only in the case where `v_stall` and `hold_in` are both low, and setting it to `DEFAULT_TRANSIENT_OUTPUT` otherwise.

```
architecture Behavioral of memory is
	signal pending_input: previous_stage_output_type := DEFAULT_PREVIOUS_STAGE_OUTPUT;
begin
	hold_out <= pending_input.valid;

	process(clk)
		variable v_input: previous_stage_output_type;
		signal pending_input: previous_stage_output_type := DEFAULT_PREVIOUS_STAGE_OUTPUT;
	begin
		if rising_edge(clk) then
			if pending_input.valid = '1' then
				v_input := pending_input;
			else
				v_input := input;
			end if;

			if hold_in = '0' then
				-- set output, v_transient, and v_stall as a function of v_input (and possibly some state or other inputs)
			end if;

			if hold_in = '0' and v_stall = '0' then
				pending_input <= DEFAULT_PREVIOUS_STAGE_OUTPUT;
				transient_out <= v_transient;
			else
				pending_input <= v_input;
				transient_out <= DEFAULT_TRANSIENT_OUTPUT;
			end if;
		end if;
	end process;
end Behavioral;
```