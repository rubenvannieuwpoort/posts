# Button debouncing

If you have any kind of digital system that reads the state of a mechanical button, you may run into the issue of "button bouncing": When the user presses or releases the button, the system detects it as a very rapid succession of a press, release, press, ... This can either be due to the button physically bouncing, or due to electrical noise. There is a standard solution called "button debouncing". It is not super complicated, but not completely trivial either, so I consider it worthy of documenting here for future reference.

The idea is that we consider the state of the button as "unknown" (or "unchanged", so equal to whatever it was before) until it is stable for a certain amount of time (meaning that the electrical input we read from the device has not changed in this time). I recommend to start with 50 milliseconds and adjust according to your needs.

With polling, we get something like:
```
global time last_state_change_time = 0;
global duration required_noise_free_time = 50ms;
global state last_state = NOT_PRESSED, denoised_state = NOT_PRESSED;

(in some loop that gets executed very often):
	current_state = read_button_state(button)
	var time = now();
	if (current_state != last_state) {
		last_state_change_time = time;
		last_state = current_state;
	}
	else if (time >= last_state_change_time + required_noise_free_time) {
		denoised_state = last_state;
	}
```

With interrupts, the solution is much cleaner:
```
global duration required_noise_free_time = 50ms;
global state last_state = NOT_PRESSED, denoised_state = NOT_PRESSED;

on state change:
  last_state = current_state;
  reset(timer, required_noise_free_time);

when timer times out:
  denoised_state = last_state;
```
