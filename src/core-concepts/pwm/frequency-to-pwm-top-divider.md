# TOP and Divider Finder for the Target Frequency

In MicroPython, you can set the PWM frequency directly without manually calculating the TOP and divider values. Internally, MicroPython computes these values from the target frequency and the system clock.

I wanted to see if there was something similar available in Rust. While discussing this in the rp-rs Matrix chat, 9names ported the [relevant C code](https://github.com/micropython/micropython/blob/634125820744efa33679fb95a6e441dadaa4f6a7/ports/rp2/machine_pwm.c#L212C13-L212C36) from MicroPython that calculates TOP and divider values into Rust. This code takes the target frequency and source clock frequency as input and gives us the corresponding TOP and divider values. You can find that implementation [in 9names Rust playground](https://github.com/9names/rp2040_rust_playground/tree/main/rp2040-pwm-freq).

You can use that Rust code directly in your own project. I compiled the same code to WASM and built a small form around it so that you can try it out here.

By default, the source clock frequency is set to the RP2350 system clock frequency of 150 MHz, and the target frequency is set to 50 Hz. You can change both values if needed.

<style>
  .pwm-card {
    max-width: 520px;
    padding: 24px;
    border-radius: 12px;
    border: 1px solid #d5c4a1;
    background: #fbf1c7;
    color: #3c3836;
    font-family: system-ui, -apple-system, BlinkMacSystemFont, sans-serif;
  }

  @media (prefers-color-scheme: dark) {
    .pwm-card {
      background: #282828;
      border-color: #504945;
      color: #ebdbb2;
    }
  }

  .pwm-card p {
    margin: 0 0 20px;
    font-size: 14px;
    color: #7c6f64;
  }

  @media (prefers-color-scheme: dark) {
    .pwm-card p {
      color: #a89984;
    }
  }

  /* Field */
  .pwm-card .pwm-field {
    display: flex;
    flex-direction: column;
    gap: 6px;
    margin-bottom: 16px;
  }

  .pwm-card .pwm-field label {
    font-size: 14px;
    color: #665c54;
  }

  @media (prefers-color-scheme: dark) {
    .pwm-card .pwm-field label {
      color: #bdae93;
    }
  }

  .pwm-card .pwm-field input {
    font-size: 16px;
    padding: 10px 12px;
    border-radius: 8px;
    border: 1px solid #d5c4a1;
    background: #f9f5d7;
    color: #3c3836;
  }

  @media (prefers-color-scheme: dark) {
    .pwm-card .pwm-field input {
      background: #1d2021;
      border-color: #504945;
      color: #ebdbb2;
    }
  }

  .pwm-card .pwm-field input:focus {
    outline: none;
    border-color: #b57614;
  }

  /* Button */
  .pwm-card .pwm-button {
    width: 100%;
    margin-top: 8px;
    padding: 12px;
    font-size: 15px;
    font-weight: 600;
    border-radius: 10px;
    border: none;
    background: #d79921;
    color: #282828;
    cursor: pointer;
  }

  .pwm-card .pwm-button:hover {
    background: #b57614;
  }

  /* Error */
  .pwm-card .pwm-error {
    margin-top: 16px;
    padding: 12px;
    border-radius: 8px;
    background: #fee2e2;
    color: #7f1d1d;
    font-size: 14px;
  }

  @media (prefers-color-scheme: dark) {
    .pwm-card .pwm-error {
      background: #3c1f1f;
      color: #fca5a5;
    }
  }

  /* Output */
  .pwm-card .pwm-output {
    margin-top: 24px;
    border-top: 1px solid #d5c4a1;
    padding-top: 16px;
  }

  @media (prefers-color-scheme: dark) {
    .pwm-card .pwm-output {
      border-top-color: #504945;
    }
  }

  .pwm-card .pwm-output h3 {
    margin: 0 0 12px;
    font-size: 16px;
    color: #7c6f64;
  }

  @media (prefers-color-scheme: dark) {
    .pwm-card .pwm-output h3 {
      color: #d5c4a1;
    }
  }

  /* Grid */
  .pwm-card .pwm-grid {
    display: grid;
    grid-template-columns: 1fr auto;
    row-gap: 10px;
    column-gap: 16px;
    font-size: 15px;
  }

  .pwm-card .pwm-key {
    color: #665c54;
  }

  @media (prefers-color-scheme: dark) {
    .pwm-card .pwm-key {
      color: #bdae93;
    }
  }

  .pwm-card .pwm-value {
    font-family: ui-monospace, SFMono-Regular, Menlo, monospace;
    font-weight: 600;
    color: #af3a03;
  }

  @media (prefers-color-scheme: dark) {
    .pwm-card .pwm-value {
      color: #fe8019;
    }
  }
</style>

<div class="pwm-card">
  <p>TOP and divider calculation</p>
  <div class="pwm-field">
    <label>Source clock (Hz)</label>
    <input id="src" value="150000000">
  </div><div class="pwm-field">
    <label>Target frequency (Hz)</label>
    <input id="freq" value="50">
  </div>
  <button id="btn" class="pwm-button">Calculate</button>
  <div id="error" class="pwm-error" hidden></div>
  <div id="out" class="pwm-output" hidden>
    <h3>Result</h3>
    <div class="pwm-grid">
      <div class="pwm-key">TOP</div><div class="pwm-value" id="o-top"></div>
      <div class="pwm-key">Divider (integer)</div><div class="pwm-value" id="o-div-int"></div>
      <div class="pwm-key">Divider (fraction)</div><div class="pwm-value" id="o-div-frac"></div>
      <div class="pwm-key">Divider (joined)</div><div class="pwm-value" id="o-div-joined"></div>
      <div class="pwm-key">Actual frequency (rounded to 5 decimal places)</div><div class="pwm-value" id="o-actual"></div>
    </div>
  </div>
</div>

> [!Note]
> The divider is shown as an integer part and a fractional part.
>
> The fractional value is **not** a decimal fraction. It represents a [4-bit fixed-point fraction](https://blog.implrust.com/posts/2025/12/fixed-point-crate-in-rust/).
>
> The effective divider is:
>
> `DIV = DIV_INT + (DIV_FRAC / 16)`
>
> For example, `DIV_INT = 45` and `DIV_FRAC = 13` means the divider is `45 + 0.8125`, not `45.13`.

## Code

If you are using rp-hal, you set the integer and fractional parts separately, like this:

```rust
pwm.set_top(65483);
pwm.set_div_int(45);
pwm.set_div_frac(13);
```

If you are using `embassy-rp`, both parts are combined into a single divider field inside the `Config` struct. Nope, this is not a floating-point value. Internally, it uses a fixed-point number to represent the integer and fractional parts together. If you are not familiar with fixed-point numbers, I have a separate blog post [Working with Fixed-Point Numbers in Rust Using the fixed Crate](https://blog.implrust.com/posts/2025/12/fixed-point-crate-in-rust/) explaining them in detail.

If you only need an integer divider, you can simply convert a `u8` value:

```rust
let mut servo_config: PwmConfig = Default::default();
servo_config.top = 46_874;
servo_config.divider = 64.into();
```

If you also want a fractional part, you need to add the `fixed` crate as a dependency and construct the divider using a fixed-point type:

```rust
let mut servo_config: PwmConfig = Default::default();
servo_config.top = 65483;
servo_config.divider = FixedU16::<U4>::from_num(45.8125);
// or
// servo_config.divider = fixed::types::U12F4::from_num(45.8125);
```

<script type="module">
  import init, { calculate_pwm } from "./assets/pwm_freq_top.js";
  await init();

  const out = document.getElementById("out");
  const errorBox = document.getElementById("error");

  const oTop = document.getElementById("o-top");
  const oDivInt = document.getElementById("o-div-int");
  const oDivFrac = document.getElementById("o-div-frac");
  const oActual = document.getElementById("o-actual");
  const oDivJoined = document.getElementById("o-div-joined");

  document.getElementById("btn").onclick = () => {
    const src = Number(document.getElementById("src").value);
    const freq = Number(document.getElementById("freq").value);

    errorBox.hidden = true;
    out.hidden = true;

    try {
        const r = calculate_pwm(src, freq);

        oTop.textContent = r.top;
        oDivInt.textContent = r.div_int;
        oDivFrac.textContent = r.div_frac;
        oActual.textContent = r.actual_freq.toFixed(5) + " Hz";

        const joinedDiv = r.div_int + r.div_frac / 16;
        oDivJoined.textContent = joinedDiv.toFixed(4);

        out.hidden = false;
    } catch (e) {
        // console.log(e);
        let message = "Unknown error";

        if (String(e).includes("FreqTooLarge")) {
            message =
            "Target frequency is too high for the PWM hardware. Try a lower frequency.";
        } else if (String(e).includes("FreqTooSmall")) {
            message =
            "Target frequency is too low for the PWM hardware. Try a higher frequency.";
        }

        errorBox.textContent = message;
        errorBox.hidden = false;
    }
  };
</script>
