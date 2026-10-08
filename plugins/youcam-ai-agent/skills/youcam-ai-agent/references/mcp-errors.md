# MCP errors

Use these recovery rules for YouCam connector failures.

- **Authentication required or expired:** Ask the user to connect or reconnect
  the YouCam connector. Never request or expose an access token. Retry only
  after OAuth succeeds.
- **Insufficient credits** (`error_type=insufficient_credit_error`,
  `error_code=VL-UN-NA-07`, or upstream type
  `Validation.Internal.InsufficientCredit`): Do not retry the failed request.
  Tell the user they have reached their usage limit and must check their plan
  or top up credits before continuing. Do not mislabel this as an
  authentication, upload, or network error.
- **Upload initialization rejected:** Correct the filename, exact byte size, or
  MIME type and initialize a new upload.
- **Byte upload failed:** Retry the same upload URL once when the failure is
  transient and the upload has not expired. Otherwise initialize a new upload.
- **Byte upload failed:** Do not call `create-beauty-content` with a reserved key
  until PUT succeeds. Initialize and upload the file again when needed.
- **Invalid tool arguments:** Correct the MCP arguments locally. Do not surface
  internal field names unless the user must change an attachment.
- **Processing still running:** This is not an error. When the MCP App rendered,
  end the current assistant turn and let the widget retrieve the result. When
  no app rendered, call `get-beauty-and-image-result` sequentially with the returned session
  and transaction IDs. Never submit the original request again.
- **Widget reports a terminal processing error:** Explain the reported failure.
  Do not create a duplicate generation or request another upload without the
  user's approval.
- **Upstream or network failure:** Retry one clearly transient failure. If it
  persists, state that YouCam is unavailable and offer a later retry or clearly
  labeled general guidance.
- **Invalid result resource:** Preserve the textual specialist answer and
  explain that the generated file could not be opened.
