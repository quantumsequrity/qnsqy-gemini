# QNSQY for Gemini CLI and Antigravity

QNSQY is a post-quantum cryptography (PQC) toolkit from Quantum Sequrity. This
extension lets Gemini CLI and Antigravity CLI use the QNSQY program installed on
your own computer. With it, the agent can:

*   find quantum-vulnerable cryptography and write a CBOM (cryptography bill of
    materials);
*   encrypt files for a recipient's public key with ML-KEM;
*   export your public key;
*   check ML-DSA signatures, QNSQY release signatures, hashes and encrypted
    files.

The tools have no field for a password, passphrase, private key, API key or
payment token, refuse any call that adds one, and never return one.

## Requirements

*   **Linux** (x86_64 or ARM64) with QNSQY's current release installed from our
    signed packages at https://quantumsequrity.com/download, so that `qnsqy` is
    on your `PATH`. The extension starts whatever `qnsqy` your `PATH` finds
    first, so install QNSQY only from there.
    Windows and macOS are not supported yet.
*   **Gemini CLI or Antigravity CLI** on the same computer. Start it from a
    normal install, not with `npx`: inside a project folder, npx puts that
    folder's `node_modules/.bin` first on your `PATH`, so a `qnsqy` placed
    there would start instead of yours.
*   **A QNSQY account.** Sign in with the QNSQY app, or with `qnsqy login` in
    a separate terminal window.

## Install

Gemini CLI:

```
gemini extensions install https://github.com/quantumsequrity/qnsqy-gemini
```

Antigravity CLI:

```
agy plugin install https://github.com/quantumsequrity/qnsqy-gemini
```

Then start a new session.

This is the official repository. Check the spelling, `quantumsequrity`,
before you install: we publish no other QNSQY extension or plugin for these
CLIs.

### Remove a QNSQY server that you added yourself

If you added QNSQY to the CLI yourself before, remove that entry. It is an MCP
server that runs `qnsqy --mcp`, and it may be called `qnsqy`. Until you remove
it, its tools with the same names as the strict ones stay visible to the agent,
and some of them take a password, such as its encrypt tool. The extension's
instructions tell the agent to use no QNSQY tool at all while such a server
runs, but they are instructions to the model, not a block.

*   **Gemini CLI**: `gemini mcp list` shows it. Remove it with
    `gemini mcp remove -s user <name>` if you added it for your user, or with
    `gemini mcp remove <name>` in the project folder where you added it.
*   **Antigravity CLI**: `agy mcp list` shows it, and `agy mcp remove <name>`
    removes it. Your own servers are in `~/.gemini/config/mcp_config.json`.
    Leave the plugin's own `mcp_config.json` as it is.

## What the extension runs, sends and fetches

*   **Runs**: the CLI starts your installed QNSQY as a local tool server named
    `qnsqy-strict`, with `qnsqy --mcp-profile=strict`. Each tool call runs the
    matching QNSQY command on your computer, inside QNSQY's sandbox.
*   **Tools**: the agent sees 13 tools (10 on the Free plan):
    *   scan;
    *   encrypt and batch encrypt;
    *   export a public key;
    *   verify, batch verify and verify a release;
    *   hash, check a hash, and check an encrypted file;
    *   status, usage and version.

    Batch encrypt, batch verify and key export need the Pro plan or above.
*   **Blocks, in Gemini CLI**: the extension hides the QNSQY tools that the
    strict profile does not have, such as decrypt and sign, from every server
    (checked with Gemini CLI 0.64). Gemini CLI lists them when you install the
    extension. Another QNSQY
    server's tools with the same names as the strict ones stay visible, so
    remove that server (see Install). The extension also blocks shell
    commands that run `qnsqy` (see Limits).
*   **Sends**: the extension itself sends nothing. QNSQY talks to our billing
    server, `billing.quantumsequrity.com`, to check your plan and record usage,
    as section 2 of our [privacy policy](https://quantumsequrity.com/privacy)
    describes:
    *   **At sign-in and at each check-in**: a device ID derived on your
        device, your computer's name and the operating system type.
    *   **Before a metered operation**: the operation type, the data size, the
        program version and the chosen algorithms.
    *   **After a metered operation**: whether it succeeded, the bytes processed
        and how long it took.
    *   **For every operation**: a usage receipt with the operation type, data
        size, time, result, credits used and a hash of the file's path. The path
        itself is not sent, but a path that is easy to guess can be checked
        against its hash.
    *   **Audit forwarding**: if you have set up audit log forwarding in
        QNSQY's settings, audit events also go to the syslog server you
        configured.

    QNSQY never sends your files, their contents, your passwords or your keys.
*   **Environment**: QNSQY's commands receive only these QNSQY variables:
    *   `QNSQY_API_KEY`, `QNSQY_CONFIG_DIR`, `QNSQY_LOG_LEVEL`,
        `QNSQY_REQUIRE_INTEGRITY` and `QNSQY_LICENSE_DATA`, if you have set
        them;
    *   `QNSQY_MACHINE_KEY`, which QNSQY sets itself for its own commands.

    They never receive a payment token or remote-storage keys.
*   **The model sees** what you ask and what the tools return: file names, paths
    and results. All of this is part of your conversation with the model's
    provider, which is why the tools never take secrets. Your plan's terms
    decide how the provider keeps and uses it. Do not point the agent at files
    whose names, paths or results your organization may not share with that
    provider.
*   **Fetches**: nothing. The extension downloads no code.

## Passwords, passphrases and keys

*   **No secret fields.** The tools have no field for a password, passphrase,
    private key, API key or payment token. QNSQY refuses any call that adds one
    before doing anything.
*   **Encryption.** Encryption through the extension is to a public key file.
*   **Steps that need a password.** To decrypt, sign, create keys or sign in,
    use the QNSQY app from your desktop menu, or run `qnsqy` in a separate
    terminal window, outside the CLI. QNSQY asks for the password in its own
    window or prompt. Create your account at https://quantumsequrity.com or in
    the app, and manage API keys in your dashboard.

## Limits

*   **Tool calls can be steered.** Text that the agent reads, for example inside
    a scanned file or a web page, can try to steer it. The strict tools refuse
    unlisted fields, including every secret field, and cannot overwrite or
    delete files. A valid call is still possible, such as encrypting a file to
    the wrong public key. Check each QNSQY call before you approve it:
    *   **Gemini CLI** asks before each call, because the extension does not
        mark its server as trusted. It stops asking for a tool once you choose
        to always allow it, and it does not ask if your settings trust the
        server or you run it in YOLO mode.
    *   **Antigravity CLI** asks before MCP calls by default, unless your
        permission settings or execution mode allow them, or you start it with
        `--dangerously-skip-permissions`.
*   **The agent's own shell.** The agent could also run `qnsqy` in its shell,
    outside the strict tools. A password on such a command line, for example
    one piped into `--password-stdin`, reaches the chat. Never approve a
    `qnsqy` command that carries a password.
    *   **Gemini CLI**: the extension blocks shell commands that run `qnsqy`,
        including piped and chained ones, so the agent has to use the strict
        tools. This is a guard against mistakes, not a security boundary: a
        command can hide the name, and your own allow rules in Gemini CLI take
        precedence over it.
    *   **Antigravity CLI**: plugins cannot block shell commands. You can add
        the same guard yourself with `/permissions`: a deny rule for
        `command(qnsqy)`.
*   **Shell mode.** In Gemini CLI, a command that you type after `!` runs in
    the CLI's shell mode, and the command and its output become part of the
    conversation. In both CLIs, run every `qnsqy` command that asks for a
    password in a separate terminal window.
*   **Project settings.** A project's own settings can add MCP servers, or
    replace one of the same name, including `qnsqy-strict`. In Gemini CLI,
    keep folder trust on (it is on by default), and trust a folder only after
    you have checked its settings.
*   **Typed secrets.** If you type a password into the chat itself, it is part of
    the conversation, so don't.

## Example prompts

1.  "Scan ~/projects/billing for quantum-vulnerable cryptography and write a
    CycloneDX CBOM to cbom.json."
2.  "Encrypt report.pdf for the public key in alice-enc.pub."
3.  "Export my public key named work to work.pub so that I can send it to a
    colleague."
4.  "Check the signature on contract.qs with bob-sign.pub, then hash
    invoice.pdf with BLAKE3."
5.  "Which QNSQY plan am I on, and how much have I used?"

## Troubleshooting

*   **No QNSQY tools appear, or the server failed to start.** One of these is
    the cause:
    *   QNSQY is not installed;
    *   `qnsqy` is not on your `PATH`;
    *   your QNSQY is older than the current release. Older releases do not know
        the strict option, so they stop.

    Install the current release from https://quantumsequrity.com/download, then
    restart the CLI. On Windows and macOS QNSQY's strict mode is not available
    yet.
*   **The agent says that another QNSQY server is running**: remove it as
    described under Install, then restart the CLI.
*   **"Not logged in"** or **"Login required"**: sign in with the QNSQY app,
    or with `qnsqy login` in a separate terminal window. Scanning needs a
    signed-in account, on every plan.
*   **A plan limit message**: some algorithms, and the batch and key-export
    tools, need the Pro or Business plan (https://quantumsequrity.com/pricing).
*   **"Not accepted in the strict profile"**: the tools never accept passwords,
    passphrases, private keys, API keys or payment tokens. Finish that step in
    the QNSQY app or in a separate terminal window.
*   **The QNSQY app does not open on a desktop that has only X11**: the app needs
    Wayland. It opens on X11 only with `--allow-x11`, which we do not
    recommend, because X11 lets other programs read keystrokes. Use the matching
    `qnsqy` command in a separate terminal window instead, for example
    `qnsqy decrypt`.

## Security and support

*   **Certifications**: QNSQY implements the FIPS 203 (ML-KEM) and FIPS 204
    (ML-DSA) algorithms. It is not FIPS 140-validated, SOC 2 certified, FedRAMP
    authorized or HIPAA-audited.
*   **Support**: support@quantumsequrity.com
*   **Security reports**: security@quantumsequrity.com
    (https://quantumsequrity.com/security)
*   **Privacy policy**: https://quantumsequrity.com/privacy
*   **Terms**: https://quantumsequrity.com/terms

## Licence

See [LICENSE](LICENSE).
