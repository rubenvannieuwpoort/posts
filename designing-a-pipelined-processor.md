# Designing a pipelined processor

In this post I will lay down a template that has the necessary pipelining logic to implement a stage in a pipelined CPU.

I have been working on a simple in-order pipelined processor for a few months now. While there's still a lot to do, things are (finally) coming together nicely. One of the most difficult parts in the design process was to get the pipelining logic right. So I decided to write something about it, in the hope that it might help someone.

When I started working on the processor, I had the impression that designing a pipelined processor was easy, or at least not *that* hard. Well, it's not *that* hard, but only if you know how to do it (and of course I did not).

I insisted that all stages in the pipeline should be strictly synchronous. Pretty much any pipelined processor will need to handle *pipeline stalls*, in which a stage cannot process data coming from the previous stage, and needs to signal this to the previous stage. However, when a stage discovers it cannot process the output of the previous stage in clock cycle N, it can only signal this to the previous stage in clock cycle N + 1 (because the stage is synchronous). But in clock cycle N + 1 the previous stage will already output the next data. So, this data needs to be buffered somewhere.

That's essentially all there is to the pipelining logic. Like I said, it's not *that* hard, once you know how to do it. However, it took me a long time to have a clear conceptual picture of the problem. I went over many iterations of writing something, testing it, and discovering it did not work. On top of that, I could not find much literature on the topic. I slowly iterated toward a design where the data of the previous stage would be buffered in case of a pipeline stall, but was not sure this was the right approach. Finally, a blog post [TODO] explained the topic clearly, and provided a much needed sanity check that my solution was a sensible approach.


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
	function f(input: previous_stage_output_type) return stage_output_type is
	begin
		-- TODO: implement
	end function;
begin
	process(clk)
	begin
		if rising_edge(clk) then
			if hold_in = '0' then
				output <= f(input);
			end if;

			hold_out <= hold_in;
		end if;
	end process;
end Behavioral;
```

This code captures the *idea* of the `hold` signal quite well, but don't try to use this code yet, though.


## 3. Making it work

So, before I said something about buffering input, right? The code from the last section *doesn't actually work*. Or well, it will work to some extent, but data will get dropped from the pipeline when a pipeline stall occurs. Not very nice.

As I tried to explain before, if the `hold_in` signal from a stage gets raised in clock cycle N, in clock cycle N + 1 the output from the stage before it will get lost. Why? In clock cycle N the previous clock cycle can't know about the pipeline stall yet, so it will happily output data in clock cycle N + 1, which will than not get accepted by the current stage (because `hold_in` is already high for that stage).

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

	function f(input: previous_stage_output_type) return stage_output_type is
	begin
		if input.valid = '0' then
			return DEFAULT_STAGE_OUTPUT;
		end if;

		-- TODO: implement
	end function;
begin
	hold_out <= buffered_input.valid;

	process(clk)
		variable v_input: previous_stage_output_type;
	begin
		if rising_edge(clk) then
			if buffered_input.valid = '1' then
				v_input := buffered_input;
			else
				v_input := input;
			end if;

			if hold_in = '0' then
				output <= f(v_input);
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

	function should_stall(input: previous_stage_output_type) return boolean is
	begin
		-- TODO: implement
	end function;

	function f(input: previous_stage_output_type) return stage_output_type is
	begin
		if input.valid = '0' then
			return DEFAULT_STAGE_OUTPUT;
		end if;

		-- TODO: implement
	end function;
begin
	hold_out <= buffered_input.valid;

	process(clk)
		variable v_input: previous_stage_output_type;
		variable v_should_stall: boolean;
	begin
		if rising_edge(clk) then
			if buffered_input.valid = '1' then
				v_input := buffered_input;
			else
				v_input := input;
			end if;

			if hold_in = '0' then
				v_should_stall := should_stall(v_input)
				if v_should_stall then
					output <= DEFAULT_STAGE_OUTPUT;
				end if;
			end if;

			if hold_in = '0' and not(v_should_stall) then
				output <= f(v_input);
				buffered_input <= DEFAULT_PREVIOUS_STAGE_OUTPUT;
			else
				buffered_input <= v_input;
			end if;
		end if;
	end process;
end Behavioral;
```

This finally is a template that is sufficient for most stages.
