---
name: youcam-ai-agent
description: Delegate beauty, skincare, makeup, hair, fashion, personal appearance, photo editing, and AI image or video requests to Perfect Corp's YouCam AI Agent MCP connector. Use for advice, recommendations, visual analysis, virtual styling, generation, or transformation in those domains, including requests with attached images. Do not use for clinical dermatology, diagnosis, medication, severe reactions, or other medical concerns.
---

# YouCam AI Agent

Delegate matching requests to the YouCam AI Agent connector. Preserve the
user's intent and rely on the specialist response instead of answering from
general knowledge while the connector is available.

## Safety boundary

Do not call YouCam for clinical dermatology or medical concerns such as a
suspected infection, severe reaction, diagnosis, treatment, or medication
interaction. Recommend an appropriate clinician and provide only clearly
labeled, non-medical context.

## Start a workflow

Call `perfect-mandatory-init` once at the start of a YouCam workflow when that
tool is available, then follow its current routing and upload guidance.

Send text-only requests to `create-beauty-content`:

- Put the user's request in `user_message` with only minimal clarification from
  recent context.
- Omit optional fields instead of passing `null`.
- Omit `session_id` on the first call.
- Do not broaden, editorialize, or add goals the user did not request.

Do not issue concurrent `create-beauty-content` calls in the same conversation.

## Handle attached files

Do not use the ChatGPT-specific `image_file` input.

When `open-beauty-upload` is available, call it with the user's original
request and current YouCam `session_id`, if any. It opens an MCP App where the
user selects files. Access attachments only through the widget and MCP tools;
shell commands and external HTTP clients are prohibited in this flow.
Wait for the widget to send a follow-up message, then call
`read_widget_context` to read the uploaded `fileKeys`, matching `fileIds`, and
original request. Call `create-beauty-content` with them as aligned `file_keys` and
`file_ids` so photo virtual try-on can resolve the original image.

On the first turn after the widget follow-up, call `read_widget_context` before
choosing or calling the next tool. The visible follow-up message only signals
that the upload finished; it is not supposed to contain file references. Do
not infer that the upload failed because those references are absent from the
message. Call the `next_tool` returned by `read_widget_context` with its
complete `arguments` object.

Only when the client cannot render MCP Apps or `open-beauty-upload` is absent,
use `asset-initialize-file-upload`, then upload the raw bytes to its `uploadUrl`.
Only after PUT succeeds, use the `fileKey` and `fileId` returned by initialize.

Call `create-beauty-content` with all collected keys in `file_keys` and all available
matching IDs in `file_ids`. Keep both lists in the same order. Never put local
paths, upload URLs, file keys, or file IDs inside `user_message`. Do not invent
a file reference when the user has not attached a usable file.

For large batches, ask the user to prioritize a small set or process them in
sequential follow-up turns.

## Recommend without a photo

For a personalized visual skincare, skin-analysis, face-analysis, or shade-
finder request that needs a newly captured face photo, call
`open-skin-analysis-camera` when available. It immediately opens a camera in
the widget; the user does not upload, attach, select, or provide a separate
photo. Never describe the widget's internal photo transfer as a user upload. If
a brief pre-tool explanation is necessary, say that you are opening the camera
now and that the user can take the photo in the camera widget. Do not repeat
photo-quality instructions outside the widget.

Once the camera app renders, end the current assistant turn without calling
another tool. The widget captures the photo, transfers it internally, runs the
file handoff, and sends a new message. On that new turn, call
`read_widget_context` first and call its returned `next_tool` exactly once with
the complete `arguments` object. Do not ask for another photo or reopen the
camera. The normal beauty widget owns processing and result presentation after
that tool call. Do not use this camera tool for makeup live VTO, which
continues through `create-beauty-content` and its own live camera.

For other requests with no usable photo, do not block a recommendation request
by asking the user to upload one first. Call `create-beauty-content` with the text-
only request so the specialist can provide a general recommendation. After
relaying the result, briefly mention that the user may upload a clear face photo
for more personalized visual analysis. Present the upload as optional, not as a
prerequisite.

Only when the user chooses to upload a photo, recommend natural light, a
centered face with hairline and chin visible, and no filters, heavy makeup,
masks, or large glasses.

## Retry an unusable face photo

Whenever a specialist result says the uploaded face photo is too small, blurry,
obstructed, poorly lit, or otherwise unusable and asks for a clearer photo,
complete these steps:

1. Reuse an existing `open-beauty-upload` widget when it is still available
   and its stored request still matches the current task.
2. Call `open-beauty-upload` with the current request and `session_id` when the
   user introduces a new Claude attachment, the request has changed, or a new
   widget is otherwise more appropriate. Do not force reuse of a widget whose
   stored `user_message` is stale.

After either widget sends its next follow-up, call `read_widget_context` and
use the latest returned `next_tool` and complete `arguments`. A generic
`status=processing` result does not trigger this retry.

## Continue a session

Save the `sessionId` returned by `create-beauty-content`. Pass it as `session_id` on
later YouCam calls in the same user workflow. Do not reuse it for unrelated
requests.

The server retains images already uploaded to or generated by that session. If
the user refers to one semantically, such as "add XXX to the image you just
generated," prefer passing that instruction in `user_message` with the existing
`session_id`. Include image fields when the user introduces a new image or when
the existing session context is insufficient.

Photo-based makeup VTO is an exception to semantic session reuse. To apply
makeup to a specific photo, the current `create-beauty-content` call needs exactly
one matching `file_keys`/`file_ids` pair, including when reusing a previously
uploaded photo. A `session_id` alone does not populate `vtoInputImageFileId`;
without that image input, the widget opens live camera try-on.

Inspect the model-visible `create-beauty-content` result before deciding who presents
the final answer:

- When the completed result requests a replacement face photo, follow the
  unusable-photo retry workflow above instead.
- For any other `status=completed` result with an `answer`, relay that complete
  answer faithfully in the assistant response even if the MCP App rendered. A
  short follow-up suggestion alone is not a substitute for the specialist
  answer.
- When it returns `status=processing` and the MCP App rendered, treat the call as
  terminal for the current assistant turn because the widget retrieves and
  renders the later `get-beauty-and-image-result` response.

When no MCP App is rendered, such as in Claude Code, inspect the tool result. If
it returns `status=processing`, call `get-beauty-and-image-result` sequentially with the
returned `sessionId` as `session_id` and the returned `transaction_id` until a
completed or error result is returned. Do not retry `create-beauty-content`, ask the
user to upload again, narrate missing polling capabilities, or propose another
workaround for the pending job.

## Present results

When a model-visible completed result is returned, treat the specialist's
`answer` as authoritative:

- Relay it faithfully without summarizing, reordering, or merging it with
  independent advice.
- Translate it into the user's language when necessary while preserving detail
  and structure.
- Preserve every result image or video resource/link and its order.
- Present `suggestedNextActions` after the answer when useful.
- Ask at most one brief, clearly separated follow-up question.

Read [MCP errors](references/mcp-errors.md) when authentication, upload,
processing, credit-limit, or upstream calls fail.
