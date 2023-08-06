# Designing a pipelined processor

I have been working on a simple in-order pipelined processor for a few months now. While there's still a lot to do, things are (finally) coming together nicely. One of the most difficult parts in the design process was to get the pipelining logic right. So I decided to document the pipelining logic for myself. Maybe it is useful for someone else as well? The template I made can be found [on github](https://github.com/rubenvannieuwpoort/stage_template).

Pretty much any pipelined processor will need to handle *pipeline stalls*. In case of a pipeline stall a stage cannot process data coming from the previous stage, and needs to signal this to the previous stage. However, for speed, all stages are *synchronous*, meaning that the output in clock cycle N depends on the state in clock cycle N - 1. So, if a stage can't process the output of the previous stage in clock cycle N, it can only tell the previous stage in clock cycle N + 1. But in clock cycle N + 1, the previous stage has already pushed new output. The essential idea is that this new output needs to be buffered by the current stage.

That's essentially all there is to the pipelining logic. If you have a clear conceptual understanding of the problem, it's not that hard. As an inexperienced hardware engineer, it took me a long time to get there. Over many iterations the idea formed to buffer the output from the previous stage in case of a pipeline stall, but I was not confident this was the right approach. I could not find any discussion of this problem online. Finally, I found the post ["Strategies for pipelining logic"](https://zipcpu.com/blog/2017/08/14/strategies-for-pipelining.html), which gives a conceptually clear explanation, and provided a much-needed sanity check.

Since the whole template might be a bit hard to understand in one go, I will iterate from the simplest possible pipeline stage and add the logic as we go, explaining each step.


## 1. Pipelining without stalls

If we have an application that does not need to implement pipeline stalls, the pipelining logic is blissfully simple. Input goes in, and the next clock cycle the associated output comes out.

```
entity stage_template is
	port(
		clk: in std_logic;
		input: in previous_stage_output_type;
		output: out stage_output_type := DEFAULT_STAGE_OUTPUT
	);
end stage_template;

architecture Behavioral of stage_template is
	function f(input: previous_stage_output_type) return stage_output_type is
	begin
		-- TODO: implement
	end function;
begin
	process(clk)
	begin
		if rising_edge(clk) then
			output <= f(input);
		end if;
	end process;
end Behavioral;
```


## 2. Introducing pipeline stalls

To handle pipeline stalls, we introduce a `hold` signal. The `hold_out` signal of a pipeline stage should be connected to the `hold_in` input of the *previous* stage, to signal that the current stage is not able to handle any more data.

If the `hold_in` input of a stage gets raised, the stage should response by "freezing". The output should not change as long as the `hold_in` signal is high. This way, the next stage has the time to wait until it is ready to start processing data again. (This could also be done by not "freezing" the output, but it is both easy and faster to keep the output the same -- clearing it and setting it again when the `hold_in` input goes low would cost us an extra cycle.)

The logic still looks pretty simple.

```
entity stage_template is
	port(
		clk: in std_logic;
		input: in previous_stage_output_type;
		output: out stage_output_type := DEFAULT_STAGE_OUTPUT;
		hold_in: in std_logic;
		hold_out: out std_logic
	);
end stage_template;

architecture Behavioral of stage_template is
begin
	process(clk)
	begin
		if rising_edge(clk) then
			if hold_in = '0' then
				-- TO DO: set output based on input
			end if;

			hold_out <= hold_in;
		end if;
	end process;
end Behavioral;
```

This code captures the *idea* of the `hold` signal quite well, but don't try to use this code yet, though.


## 3. Making it work

So, before I said something about buffering input, right? The code from the last section *doesn't actually work*: Data will get dropped from the pipeline when a pipeline stall occurs.

As mentioned before, if the `hold_in` signal from a stage gets raised in clock cycle N, in clock cycle N + 1 the output from the stage before it will get lost. Why? In clock cycle N the previous clock cycle can't know about the pipeline stall yet, so it will happily output data in clock cycle N + 1, which will than not get accepted by the current stage (because `hold_in` is already high for that stage).

This change is a bit more complicated. I have added a `valid` signal to the input. If it is zero, the data is not valid and can be ignored. The input is now chosen from either the buffered input or the actual input to the stage; if `buffered_input` is valid, the variable `v_input` is taken as the input, otherwise the input to the stage (e.g. the `input` signal) is taken as the input.

The `buffered_input` register is cleared if `hold_in` is low. This works because in this case the input will be selected from `buffered_input`, and in the next clock cycle the output will be generated from `buffered_input`, so it's no longer needed.

Another trick that is applied is that the `hold_out` signal is raised if and only if `buffered_input` holds a valid instruction. This way, the flow of data into the stage will keep going until the stage can't process any more data, instead of just completely locking up one cycle after the `hold_in` signal is raised.

```
entity stage_template is
	port(
		clk: in std_logic;
		input: in previous_stage_output_type;
		output: out stage_output_type := DEFAULT_STAGE_OUTPUT;
		hold_in: in std_logic;
		hold_out: out std_logic
	);
end stage_template;


architecture Behavioral of stage_template is
	signal buffered_input: previous_stage_output_type := DEFAULT_PREVIOUS_STAGE_OUTPUT;
begin
	hold_out <= buffered_input.valid;

	process(clk)
		variable v_input: previous_stage_output_type;
	begin
		if rising_edge(clk) then
			-- input selection
			if buffered_input.valid = '1' then
				v_input := buffered_input;
			else
				v_input := input;
			end if;

			if hold_in = '0' then
				-- TODO: set output based on v_input
				buffered_input <= DEFAULT_PREVIOUS_STAGE_OUTPUT;
			else
				buffered_input <= v_input;
			end if;
		end if;
	end process;
end Behavioral;
```


## 4. Generating a pipeline stall

The code in the previous section works fine, but the template there only passes on a `hold` signal. It doesn't cover how to *generate* pipeline stalls.

For this, we introduce a variable `v_stall` that is generated from the input with a function `should_stall`. If the stage stalls, it should not output any data, so the output is set to `DEFAULT_STAGE_OUTPUT`, which is has the `valid` flag set to `0`.

```
entity stage_template is
	port(
		clk: in std_logic;
		input: in previous_stage_output_type;
		output: out stage_output_type := DEFAULT_STAGE_OUTPUT;
		hold_in: in std_logic;
		hold_out: out std_logic
	);
end stage_template;

architecture Behavioral of stage_template is
	signal buffered_input: previous_stage_output_type := DEFAULT_PREVIOUS_STAGE_OUTPUT;
begin
	hold_out <= buffered_input.valid;

	process(clk)
		variable v_input: previous_stage_output_type;
		variable v_output: stage_output_type;
		variable v_should_stall: boolean;
	begin
		if rising_edge(clk) then
			-- input selection
			if buffered_input.valid = '1' then
				v_input := buffered_input;
			else
				v_input := input;
			end if;

			-- output selection
			-- TO DO: set v_output and v_should_stall based on v_input

			-- set output
			if hold_in = '0' then
				output <= v_output;
			end if;

			-- input buffering
			if hold_in = '0' and not(v_should_stall) then
				buffered_input <= DEFAULT_PREVIOUS_STAGE_OUTPUT;
			else
				buffered_input <= v_input;
			end if;
		end if;
	end process;
end Behavioral;
```

This finally is a template that is sufficient for most stages.
