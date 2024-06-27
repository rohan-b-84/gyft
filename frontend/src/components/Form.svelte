<!-- <script lang="ts">
  import type { kFormData } from "$lib/_types";
  import { quintIn } from "svelte/easing";
  import { fade } from "svelte/transition";
  import Header from "./Header.svelte";
  import Footer from "./Footer.svelte";
  import { writable } from "svelte/store";

  let securityQuestion = writable("");
  let fetchedSecurityQuestion = writable(false);
  let otpSent = writable(false);
  let loggedIn = writable(false);
  export let formData: kFormData;

  function getFormData() {
    const _formData = new FormData();
    _formData.append("roll_number", formData.roll || "");
    _formData.append("password", formData.pass || "");
    _formData.append("secret_answer", formData.secret_answer || "");
    _formData.append("otp", formData.email_otp || "");
    return _formData;
  }

  async function getSecurityQuestion() {
    try {
      const res = await fetch("http://127.0.0.1:5000/secret-question", {
        method: "POST",
        body: getFormData(),
      });

      if (!res.ok) {
        const errorData = await res.json();
        throw new Error(
          errorData.message || "Failed to fetch security question"
        );
      }

      const data = await res.json();
      securityQuestion.set(data.secret_question);
      localStorage.setItem("jwt", data.jwt); // Store JWT in localStorage
      fetchedSecurityQuestion.set(true);
    } catch (error) {
      console.error("There was a problem with the fetch operation:", error);
    }
  }

  async function sendOTP() {
    try {
      const res = await fetch("http://127.0.0.1:5000/request-otp", {
        method: "POST",
        headers: {
          Authorization: `Bearer ${localStorage.getItem("jwt")}`,
        },
        body: getFormData(),
      });

      if (!res.ok) {
        const errorData = await res.json();
        throw new Error(errorData.message || "Failed to send OTP");
      }

      otpSent.set(true);
    } catch (error) {
      console.error("There was a problem with the fetch operation:", error);
    }
  }

  async function login() {
    try {
      const res = await fetch("http://127.0.0.1:5000/login", {
        method: "POST",
        headers: {
          Authorization: `Bearer ${localStorage.getItem("jwt")}`,
        },
        body: getFormData(),
      });

      if (!res.ok) {
        const errorData = await res.json();
        throw new Error(errorData.message || "Failed to login");
      }
      loggedIn.set(true);
    } catch (error) {
      console.error("There was a problem with the fetch operation:", error);
    }
  }

  async function downloadICSFile() {
    try {
      const res = await fetch("http://127.0.0.1:5000/download-ics", {
        method: "POST",
        headers: {
          Authorization: `Bearer ${localStorage.getItem("jwt")}`,
        },
        body: getFormData(),
      });

      if (!res.ok) {
        const errorData = await res.json();
        throw new Error(errorData.message || "Failed to download ICS file");
      }

      const blob = await res.blob();
      const url = URL.createObjectURL(blob);
      const a = document.createElement("a");
      a.href = url;
      a.download = "timetable.ics";
      document.body.appendChild(a);
      a.click();
      a.remove();
      URL.revokeObjectURL(url);

      formData.roll = "";
      formData.pass = "";
      formData.secret_answer = "";
      formData.email_otp = "";
      otpSent.set(false);
      securityQuestion.set("");
      fetchedSecurityQuestion.set(false);
    } catch (error) {
      console.error("There was a problem with the fetch operation:", error);
    }
  }
</script>

{#if !$fetchedSecurityQuestion}
  <div in:fade={{ duration: 300, delay: 0, easing: quintIn }}>
    <Header logoVisible={true} />
    <div class="input-container">
      <div>
        <h3>ERP Roll Number :</h3>
        <input
          bind:value={formData.roll}
          placeholder="Enter Your ERP Roll Number"
        />
      </div>
      <button on:click={getSecurityQuestion}>Get Security Question</button>
    </div>
    <Footer />
  </div>
{:else}
  <div in:fade={{ duration: 300, delay: 0, easing: quintIn }}>
    <Header logoVisible={false} />
    <div class="input-container">
      <div>
        <h3>ERP Roll Number :</h3>
        <input
          bind:value={formData.roll}
          placeholder="Enter Your ERP Roll Number"
        />
      </div>
      <div>
        <h3>{$securityQuestion} :</h3>
        <input
          bind:value={formData.secret_answer}
          type="password"
          placeholder="Security Answer"
        />
      </div>
      <div>
        <h3>Password :</h3>
        <input
          bind:value={formData.pass}
          type="password"
          placeholder="Password"
        />
      </div>
      {#if !$otpSent}
        <button on:click={sendOTP}>Send OTP</button>
      {:else}
        <div>
          <div>
            <h3>OTP :</h3>
            <input
              bind:value={formData.email_otp}
              type="password"
              placeholder="OTP"
            />
          </div>
          {#if !$loggedIn}
            <button on:click={login}>Login</button>
          {:else}
            <button on:click={downloadICSFile}>Download ICS File</button>
          {/if}
        </div>
      {/if}
    </div>
    <Footer />
  </div>
{/if}

<style>
  .input-container {
    display: flex;
    flex-direction: column;
    gap: 0.75rem;
    border-bottom: 2px solid white;
    box-shadow: 0 20px 10px -25px #000d;
    margin-bottom: 1rem;
    padding-bottom: 1rem;
  }
  input {
    border: none;
    background-color: white;
    padding: 1rem;
    font-size: 1rem;
    border-radius: 8px;
    width: 100%;
    max-width: 20rem;
  }
  button {
    border: none;
    background-color: #00ba29;
    padding: 1rem;
    font-size: 1rem;
    border-radius: 8px;
    width: 100%;
    max-width: 20rem;
    color: white;
    font-weight: bold;
    cursor: pointer;
    margin-inline: auto;
  }
</style> -->

<script lang="ts">
  import type { kFormData } from "$lib/_types";
  import { quintIn } from "svelte/easing";
  import { fade } from "svelte/transition";
  import Header from "./Header.svelte";
  import Footer from "./Footer.svelte";
  import { writable } from "svelte/store";

  let securityQuestion = writable("");
  let fetchedSecurityQuestion = writable(false);
  let otpSent = writable(false);
  let loggedIn = writable(false);
  let isLoading = writable(false);
  let errorMessage = writable("");
  export let formData: kFormData;

  function getFormData() {
    const _formData = new FormData();
    _formData.append("roll_number", formData.roll || "");
    _formData.append("password", formData.pass || "");
    _formData.append("secret_answer", formData.secret_answer || "");
    _formData.append("otp", formData.email_otp || "");
    return _formData;
  }

  async function getSecurityQuestion() {
    isLoading.set(true);
    errorMessage.set("");
    try {
      const res = await fetch("http://127.0.0.1:5000/secret-question", {
        method: "POST",
        body: getFormData(),
      });

      if (!res.ok) {
        const errorData = await res.json();
        throw new Error(
          errorData.message || "Failed to fetch security question"
        );
      }

      const data = await res.json();
      securityQuestion.set(data.secret_question);
      localStorage.setItem("jwt", data.jwt); // Store JWT in localStorage
      fetchedSecurityQuestion.set(true);
    } catch (error) {
      errorMessage.set(error.message);
      console.error("There was a problem with the fetch operation:", error);
    } finally {
      isLoading.set(false);
    }
  }

  async function sendOTP() {
    isLoading.set(true);
    errorMessage.set("");
    try {
      const res = await fetch("http://127.0.0.1:5000/request-otp", {
        method: "POST",
        headers: {
          Authorization: `Bearer ${localStorage.getItem("jwt")}`,
        },
        body: getFormData(),
      });

      if (!res.ok) {
        const errorData = await res.json();
        throw new Error(errorData.message || "Failed to send OTP");
      }

      otpSent.set(true);
    } catch (error) {
      errorMessage.set(error.message);
      console.error("There was a problem with the fetch operation:", error);
    } finally {
      isLoading.set(false);
    }
  }

  async function login() {
    isLoading.set(true);
    errorMessage.set("");
    try {
      const res = await fetch("http://127.0.0.1:5000/login", {
        method: "POST",
        headers: {
          Authorization: `Bearer ${localStorage.getItem("jwt")}`,
        },
        body: getFormData(),
      });

      if (!res.ok) {
        const errorData = await res.json();
        throw new Error(errorData.message || "Failed to login");
      }
      loggedIn.set(true);
    } catch (error) {
      errorMessage.set(error.message);
      console.error("There was a problem with the fetch operation:", error);
    } finally {
      isLoading.set(false);
    }
  }

  async function downloadICSFile() {
    isLoading.set(true);
    errorMessage.set("");
    try {
      const res = await fetch("http://127.0.0.1:5000/download-ics", {
        method: "POST",
        headers: {
          Authorization: `Bearer ${localStorage.getItem("jwt")}`,
        },
        body: getFormData(),
      });

      if (!res.ok) {
        const errorData = await res.json();
        throw new Error(errorData.message || "Failed to download ICS file");
      }

      const blob = await res.blob();
      const url = URL.createObjectURL(blob);
      const a = document.createElement("a");
      a.href = url;
      a.download = "timetable.ics";
      document.body.appendChild(a);
      a.click();
      a.remove();
      URL.revokeObjectURL(url);

      formData.roll = "";
      formData.pass = "";
      formData.secret_answer = "";
      formData.email_otp = "";
      otpSent.set(false);
      securityQuestion.set("");
      fetchedSecurityQuestion.set(false);
    } catch (error) {
      errorMessage.set(error.message);
      console.error("There was a problem with the fetch operation:", error);
    } finally {
      isLoading.set(false);
    }
  }
</script>

{#if !$fetchedSecurityQuestion}
  <div in:fade={{ duration: 300, delay: 0, easing: quintIn }}>
    <Header logoVisible={true} />
    <div class="input-container">
      <div>
        <h3>ERP Roll Number :</h3>
        <input
          bind:value={formData.roll}
          placeholder="Enter Your ERP Roll Number"
        />
      </div>
      <button on:click={getSecurityQuestion} disabled={$isLoading}>
        {#if $isLoading}Loading...{/if}

        {#if !$isLoading}Get Security Question{/if}
      </button>
      {#if $errorMessage}
        <div class="error-message">{$errorMessage}</div>
      {/if}
    </div>
    <Footer />
  </div>
{:else}
  <div in:fade={{ duration: 300, delay: 0, easing: quintIn }}>
    <Header logoVisible={false} />
    <div class="input-container">
      <div>
        <h3>ERP Roll Number :</h3>
        <input
          bind:value={formData.roll}
          placeholder="Enter Your ERP Roll Number"
        />
      </div>
      <div>
        <h3>{$securityQuestion} :</h3>
        <input
          bind:value={formData.secret_answer}
          type="password"
          placeholder="Security Answer"
        />
      </div>
      <div>
        <h3>Password :</h3>
        <input
          bind:value={formData.pass}
          type="password"
          placeholder="Password"
        />
      </div>
      {#if !$otpSent}
        <button on:click={sendOTP} disabled={$isLoading}>
          {#if $isLoading}Sending OTP...{/if}
          {#if !$isLoading}Send OTP{/if}


        </button>
      {:else}
        <div>
          <div>
            <h3>OTP :</h3>
            <input
              bind:value={formData.email_otp}
              type="password"
              placeholder="OTP"
            />
          </div>
          {#if !$loggedIn}
            <button on:click={login} disabled={$isLoading}>
              {#if $isLoading}Logging in...{/if}
              {#if !$isLoading}Log In{/if}
            </button>
          {:else}
            <button on:click={downloadICSFile} disabled={$isLoading}>
              {#if $isLoading}Downloading...{/if}

              {#if !$isLoading}Download ICS File{/if}
            </button>
          {/if}
        </div>
      {/if}
      {#if $errorMessage}
        <div class="error-message">{$errorMessage}</div>
      {/if}
    </div>
    <Footer />
  </div>
{/if}

<style>
  .input-container {
    display: flex;
    flex-direction: column;
    gap: 0.75rem;
    border-bottom: 2px solid white;
    box-shadow: 0 20px 10px -25px #000d;
    margin-bottom: 1rem;
    padding-bottom: 1rem;
  }
  input {
    border: none;
    background-color: white;
    padding: 1rem;
    font-size: 1rem;
    border-radius: 8px;
    width: 100%;
    max-width: 20rem;
  }
  button {
    border: none;
    background-color: #00ba29;
    padding: 1rem;
    font-size: 1rem;
    border-radius: 8px;
    width: 100%;
    max-width: 20rem;
    color: white;
    font-weight: bold;
    cursor: pointer;
    margin-inline: auto;
  }
  button:disabled {
    background-color: #009320;
    cursor: not-allowed;
  }
  .error-message {
    color: red;
    font-weight: bold;
    margin-top: 1rem;
  }
</style>
