# Changelog

All notable changes to **Pipecat** will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

<!-- towncrier release notes start -->

## [0.0.99] - 2026-01-13

### Added

- Introducing user turn strategies. User turn strategies indicate when the user
  turn starts or stops. In conversational agents, these are often referred to
  as start/stop speaking or turn-taking plans or policies.

  User turn start strategies indicate when the user starts speaking (e.g.
  using VAD events or when a user says one or more words).

  User turn stop strategies indicate when the user stops speaking (e.g. using
  an end-of-turn detection model or by observing incoming transcriptions).

  A list of strategies can be specified for both strategies; strategies are
  evaluated in order until one evaluates to true.

  Available user turn start strategies:

  - VADUserTurnStartStrategy
  - TranscriptionUserTurnStartStrategy
  - MinWordsUserTurnStartStrategy
  - ExternalUserTurnStartStrategy

  Available user turn stop strategies:

  - TranscriptionUserTurnStopStrategy
  - TurnAnalyzerUserTurnStopStrategy
  - ExternalUserTurnStopStrategy

  The default strategies are:

  - start: [VADUserTurnStartStrategy, TranscriptionUserTurnStartStrategy]
  - stop: [TranscriptionUserTurnStopStrategy]

  Turn strategies are configured when setting up `LLMContextAggregatorPair`.
  For example:

  ```python
  context_aggregator = LLMContextAggregatorPair(
      context,
      user_params=LLMUserAggregatorParams(
          user_turn_strategies=UserTurnStrategies(
              stop=[
                  TurnAnalyzerUserTurnStopStrategy(turn_analyzer=LocalSmartTurnAnalyzerV3(params=SmartTurnParams())
                  )
              ],
          )
      ),
  )
  ```

  In order to use the user turn strategies you must update to the new
  universal `LLMContext` and `LLMContextAggregatorPair`.
  (PR [#3045](https://github.com/pipecat-ai/pipecat/pull/3045))

- Added `RNNoiseFilter` for real-time noise suppression using RNNoise neural
  network via pyrnnoise library.
  (PR [#3205](https://github.com/pipecat-ai/pipecat/pull/3205))

- Added `GrokRealtimeLLMService` for xAI's Grok Voice Agent API with real-time
  voice conversations:

  - Support for real-time audio streaming with WebSocket connection
  - Built-in server-side VAD (Voice Activity Detection)
  - Multiple voice options: Ara, Rex, Sal, Eve, Leo
  - Built-in tools support: web_search, x_search, file_search
  - Custom function calling with standard Pipecat tools schema
  - Configurable audio formats (PCM at 8kHz-48kHz)
    (PR [#3267](https://github.com/pipecat-ai/pipecat/pull/3267))

- Added an approximation of TTFB for Ultravox.
  (PR [#3268](https://github.com/pipecat-ai/pipecat/pull/3268))

- Added a new `AudioContextTTSService` to the TTS service base classes. The
  `AudioContextWordTTSService` now inherits from `AudioContextTTSService` and
  `WebsocketWordTTSService`.
  (PR [#3289](https://github.com/pipecat-ai/pipecat/pull/3289))

- `LLMUserAggregator` now exposes the following events:

  - `on_user_turn_started`: triggered when a user turn starts
  - `on_user_turn_stopped`: triggered when a user turn ends
  - `on_user_turn_stop_timeout`: triggered when a user turn does not stop
    and times out
    (PR [#3291](https://github.com/pipecat-ai/pipecat/pull/3291))

- Introducing user mute strategies. User mute strategies indicate when user
  input should be muted based on the current system state.

  In conversational agents, user mute strategies are used to prevent user
  input from interrupting bot speech, tool execution, or other critical system
  operations.

  A list of strategies can be specified; all strategies are evaluated for
  every frame so that each strategy can maintain its internal state. A user
  frame is muted if any of the configured strategies indicates it should be
  muted.

  Available user mute strategies:

  - `FirstSpeechUserMuteStrategy`
  - `MuteUntilFirstBotCompleteUserMuteStrategy`
  - `AlwaysUserMuteStrategy`
  - `FunctionCallUserMuteStrategy`

  User mute strategies replace the legacy `STTMuteFilter` and provide a more
  flexible and composable approach to muting user input.

  User mute strategies are configured when setting up the
  `LLMContextAggregatorPair`. For example:

  ```python
  context_aggregator = LLMContextAggregatorPair(
      context,
      user_params=LLMUserAggregatorParams(
          user_mute_strategies=[
              FirstSpeechUserMuteStrategy(),
          ]
      ),
  )
  ```

  In order to use user mute strategies you should update to the new universal
  `LLMContext` and `LLMContextAggregatorPair`.
  (PR [#3292](https://github.com/pipecat-ai/pipecat/pull/3292))

- Added `use_ssl` parameter to `NvidiaSTTService`, `NvidiaSegmentedSTTService`
  and `NvidiaTTSService`.
  (PR [#3300](https://github.com/pipecat-ai/pipecat/pull/3300))

- Added `enable_interruptions` constructor argument to all user turn
  strategies. This tells the `LLMUserAggregator` to push or not push an
  `InterruptionFrame`.
  (PR [#3316](https://github.com/pipecat-ai/pipecat/pull/3316))

- Added `split_sentences` parameter to `SpeechmaticsSTTService` to control
  sentence splitting behavior for finals on sentence boundaries.
  (PR [#3328](https://github.com/pipecat-ai/pipecat/pull/3328))

- Added word-level timestamp support to `AzureTTSService` for accurate
  text-to-audio synchronization.
  (PR [#3334](https://github.com/pipecat-ai/pipecat/pull/3334))

- Added `pronunciation_dict_id` parameter to `CartesiaTTSService.InputParams`
  and `CartesiaHttpTTSService.InputParams` to support Cartesia's pronunciation
  dictionary feature for custom pronunciations.
  (PR [#3346](https://github.com/pipecat-ai/pipecat/pull/3346))

- Added support for using the HeyGen LiveAvatar API with the `HeyGenTransport`
  (see https://www.liveavatar.com/).
  (PR [#3357](https://github.com/pipecat-ai/pipecat/pull/3357))

- Added image support to `OpenAIRealtimeLLMService` via `InputImageRawFrame`:

  - New `start_video_paused` parameter to control initial video input state
  - New `video_frame_detail` parameter to set image processing quality
    ("auto",
    "low", or "high"). This corresponds to OpenAI Realtime's `image_detail`
    parameter.
  - `set_video_input_paused()` method to pause/resume video input at runtime
  - `set_video_frame_detail()` method to adjust video frame quality
    dynamically
  - Automatic rate limiting (1 frame per second) to prevent API overload
    (PR [#3360](https://github.com/pipecat-ai/pipecat/pull/3360))

- Added `UserTurnProcessor`, a frame processor built on `UserTurnController`
  that pushes `UserStartedSpeakingFrame` and `UserStoppedSpeakingFrame` frames
  and interruptions based on the controller's user turn strategies.
  (PR [#3372](https://github.com/pipecat-ai/pipecat/pull/3372))

- Added `UserTurnController` to manage user turns. It emits
  `on_user_turn_started`, `on_user_turn_stopped`, and
  `on_user_turn_stop_timeout` events, and can be integrated into processors to
  detect and handle user turns. `LLMUserAggregator` and `UserTurnProcessor` are
  implemented using this controller.
  (PR [#3372](https://github.com/pipecat-ai/pipecat/pull/3372))

- Added `should_interrupt` property to `DeepgramFluxSTTService`,
  `DeepgramSTTService`, and `SpeechmaticsSTTService` to configure whether the
  bot should be interrupted when the external service detects user speech.
  (PR [#3374](https://github.com/pipecat-ai/pipecat/pull/3374))

- `LLMAssistantAggregator` now exposes the following events:

  - `on_assistant_turn_started`: triggered when the assistant turn starts
  - `on_assistant_turn_stopped`: triggered when the assistant turn ends
  - `on_assistant_thought`: triggered when there's an assistant thought
    available
    (PR [#3385](https://github.com/pipecat-ai/pipecat/pull/3385))

- Added `KrispVivaTurn` analyzer for end of turn detection using the Krisp VIVA
  SDK (requires `krisp_audio`).
  (PR [#3391](https://github.com/pipecat-ai/pipecat/pull/3391))

- Added support for setting up a pipeline task from external files. You can now
  register custom pipeline task setup files by setting the
  `PIPECAT_SETUP_FILES` environment variable. This variable should contain a
  colon-separated list of Python files (e.g. `export
PIPECAT_SETUP_FILES="setup1.py:setup.py:..."`). Each file must define a
  function with the following signature:

  ```python
  async def setup_pipeline_task(task: PipelineTask):
      ...
  ```

  (PR [#3397](https://github.com/pipecat-ai/pipecat/pull/3397))

- Added a keepalive task for `InworldTTSService` to keep the service connected
  in the event of no generations for longer periods of time.
  (PR [#3403](https://github.com/pipecat-ai/pipecat/pull/3403))

- Added `enable_vad` to `Params` for use in the `GladiaSTTService`. When
  enabled, `GladiaSTTService` acts as the turn controller, emitting
  `UserStartedSpeakingFrame`, `UserStoppedSpeakingFrame`, and optionally
  `InterruptionFrame`.
  (PR [#3404](https://github.com/pipecat-ai/pipecat/pull/3404))

- Added `should_interrupt` property to `GladiaSTTService` to configure whether
  the bot should be interrupted when the external service detects user speech.
  (PR [#3404](https://github.com/pipecat-ai/pipecat/pull/3404))

- Added `VonageFrameSerializer` for the Vonage Video API Audio Connector
  WebSocket protocol.
  (PR [#3410](https://github.com/pipecat-ai/pipecat/pull/3410))

- Added `append_trailing_space` parameter to `TTSService` to automatically
  append a trailing space to text before sending to TTS, helping prevent some
  services from vocalizing trailing punctuation.
  (PR [#3424](https://github.com/pipecat-ai/pipecat/pull/3424))

### Changed

- Updated `ElevenLabsRealtimeSTTService` to accept the
  `include_language_detection` parameter to detect language.

  ```python
    stt = ElevenLabsRealtimeSTTService(
        api_key=os.getenv("ELEVENLABS_API_KEY"),
        include_language_detection=True
    )
  ```

  (PR [#3216](https://github.com/pipecat-ai/pipecat/pull/3216))

- Updated `SpeechmaticsSTTService` to use new Python Voice SDK with improved
  VAD, Smart Turn capabilities, and brings dramatic improvements to latency
  without any impact on accuracy. Use the `turn_detection_mode` parameter to control
  the endpointing of speech, with `TurnDetectionMode.EXTERNAL` (default),
  `TurnDetectionMode.ADAPTIVE`, or `TurnDetectionMode.SMART_TURN`.

  ```python
      stt = SpeechmaticsSTTService(
          api_key=os.getenv("SPEECHMATICS_API_KEY"),
          params=SpeechmaticsSTTService.InputParams(
              language=Language.EN,
              turn_detection_mode=SpeechmaticsSTTService.TurnDetectionMode.ADAPTIVE,
              speaker_active_format="<{speaker_id}>{text}</{speaker_id}>",
          ),
      )
  ```

  (PR [#3225](https://github.com/pipecat-ai/pipecat/pull/3225))

- `daily-python` updated to 0.23.0.
  (PR [#3257](https://github.com/pipecat-ai/pipecat/pull/3257))

- `TranscriptionFrame` and `InterimTranscriptionFrame` produced by
  `DailyTransport` now include the transport source (i.e., the originating
  audio track).
  (PR [#3257](https://github.com/pipecat-ai/pipecat/pull/3257))

- Updates to Inworld TTS services:

  - Improved `InworldTTSService`'s websocket implementation to better flush
    and close context to better handle long inputs.
  - Improved docstrings for `InworldTTSService` and `InworldHttpTTSService`.
    (PR [#3288](https://github.com/pipecat-ai/pipecat/pull/3288))

- Improved the error handling and reconnection logic for `WebsocketServer` by
  distinguishing between errors when disconnecting and websocket communication
  errors.
  (PR [#3392](https://github.com/pipecat-ai/pipecat/pull/3392))

- Updated `DeepgramSTTService` to push user started/stopped speaking and
  interruption frames when `vad_enabled` is set to true. This centralizes the
  frames into the service, removing the need to have your application code
  handle Deepgram's events and push these frames.
  (PR [#3314](https://github.com/pipecat-ai/pipecat/pull/3314))

- Added encoding validation to `DeepgramTTSService` to prevent unsupported
  encodings from reaching the API. The service now raises `ValueError` at
  initialization with a clear error message.
  (PR [#3329](https://github.com/pipecat-ai/pipecat/pull/3329))

- Updated `read_audio_frame` & `read_video_frame` methods in
  `SmallWebRTCClient` to check if the track is enabled before logging a
  warning.
  (PR [#3336](https://github.com/pipecat-ai/pipecat/pull/3336))

- Updated `CartesiaTTSService` to support setting `language=None`, resulting in
  Cartesia auto-detecting the language of the conversation.
  (PR [#3366](https://github.com/pipecat-ai/pipecat/pull/3366))

- The bundled Smart Turn weights are now updated to v3.2, which has better
  handling of short utterances, and is more robust against background noise.
  (PR [#3367](https://github.com/pipecat-ai/pipecat/pull/3367))

- Updated `SpeechmaticsSTTService` dependency to `speechmatics-voice[smart]>=0.2.6`
  (PR [#3371](https://github.com/pipecat-ai/pipecat/pull/3371))

- Smart Turn now takes into account `vad_start_seconds` when buffering audio,
  meaning that the start of the turn audio is not cut off. This improves
  accuracy for short utterances.

- The default value of `pre_speech_ms` is now set to 500ms for Smart Turn.
  (PR [#3377](https://github.com/pipecat-ai/pipecat/pull/3377))

- Improved Krisp SDK management to allow `KrispVivaTurn` and `KrispVivaFilter`
  to share a single SDK instance within the same process.
  (PR [#3391](https://github.com/pipecat-ai/pipecat/pull/3391))

- Updated default model for `GroqTTSService` to `canopylabs/orpheus-v1-english`
  and voice ID to `autumn`.
  (PR [#3399](https://github.com/pipecat-ai/pipecat/pull/3399))

- Enhanced `FastAPIWebsocketTransport` with optional protocol-level audio
  packetization via the `fixed_audio_packet_size` parameter to support media
  endpoints requiring strict framing and real-time pacing.
  (PR [#3410](https://github.com/pipecat-ai/pipecat/pull/3410))

- `DeepgramTTSService` and `RimeTTSService` now set `append_trailing_space` to
  `True` to prevent punctuation (e.g., “dot”) from being pronounced.
  (PR [#3424](https://github.com/pipecat-ai/pipecat/pull/3424))

- Updated `GeminiLiveLLMService` to push `LLMThoughtStartFrame`,
  `LLMThoughtTextFrame`, and `LLMThoughtEndFrame` when the model returns
  thought content.
  (PR [#3431](https://github.com/pipecat-ai/pipecat/pull/3431))

### Deprecated

- `pipecat.audio.interruptions.MinWordsInterruptionStrategy` is deprecated. Use
  `pipecat.turns.user_start.MinWordsUserTurnStartStrategy` with
  `LLMUserAggregator`'s new `user_turn_strategies` parameter instead.
  (PR [#3045](https://github.com/pipecat-ai/pipecat/pull/3045))

- `FrameProcessor.interruption_strategies` is deprecated, use
  `LLMUserAggregator`'s new `user_turn_strategies` parameter instead.
  (PR [#3045](https://github.com/pipecat-ai/pipecat/pull/3045))

- The `LLMUserAggregatorParams` and `LLMAssistantAggregatorParams` classes in
  `pipecat.processors.aggregators.llm_response` are now deprecated. Use the new
  universal `LLMContext` and `LLMContextAggregatorPair` instead.
  (PR [#3045](https://github.com/pipecat-ai/pipecat/pull/3045))

- Deprecated the `emulated` field in the `UserStartedSpeakingFrame` and
  `UserStoppedSpeakingFrame` frames.
  (PR [#3045](https://github.com/pipecat-ai/pipecat/pull/3045))

- `EmulateUserStartedSpeakingFrame` and `EmulateUserStoppedSpeakingFrame`
  frames are deprecated.
  (PR [#3045](https://github.com/pipecat-ai/pipecat/pull/3045))

- ⚠️ `TransportParams.turn_analyzer` is deprecated and might result in
  unexpected behavior, use `LLMUserAggregator`'s new `user_turn_strategies`
  parameter instead.
  (PR [#3045](https://github.com/pipecat-ai/pipecat/pull/3045))

- For `SpeechmaticsSTTService`, the `end_of_utterance_mode` parameter is
  deprecated. Use the new `turn_detection_mode` parameter instead, with
  `TurnDetectionMode.EXTERNAL`,`TurnDetectionMode.ADAPTIVE`, or
  `TurnDetectionMode.SMART_TURN`. The `enable_vad` parameter is also
  deprecated and is inferred from the `turn_detection_mode`.
  (PR [#3225](https://github.com/pipecat-ai/pipecat/pull/3225))

- `OpenAILLMContext` and its associated things (context aggregators, etc.) are
  now deprecated in favor of the universal `LLMContext` and its associated
  things.

  From the developer's point of view, switching to using `LLMContext`
  machinery will usually be a matter of going from this:

  ```python
  context = OpenAILLMContext(messages, tools)
  context_aggregator = llm.create_context_aggregator(context)
  ```

  To this:

  ```
  context = LLMContext(messages, tools)
  context_aggregator = LLMContextAggregatorPair(context)
  ```

  (PR [#3263](https://github.com/pipecat-ai/pipecat/pull/3263))

- `STTMuteFilter` is deprecated and will be removed in a future version. Use
  `LLMUserAggregator`'s new `user_mute_strategies` instead.
  (PR [#3292](https://github.com/pipecat-ai/pipecat/pull/3292))

- `FrameProcessor.interruptions_allowed` is now deprecated, use
  `LLMUserAggregator`'s new parameter `user_mute_strategies` instead.
  (PR [#3297](https://github.com/pipecat-ai/pipecat/pull/3297))

- `PipelineParams.allow_interruptions` is now deprecated, use
  `LLMUserAggregator`'s new parameter `user_turn_strategies` instead. For
  example, to disable interruptions but still get user turns you can do:

  ```python
  context_aggregator = LLMContextAggregatorPair(
      context,
      user_params=LLMUserAggregatorParams(
          user_turn_strategies=UserTurnStrategies(
              start=[TranscriptionUserTurnStartStrategy(enable_interruptions=False)],
          ),
      ),
  )
  ```

  (PR [#3297](https://github.com/pipecat-ai/pipecat/pull/3297))

- `TranscriptProcessor` and related data classes and frames
  (`TranscriptionMessage`, `ThoughtTranscriptionMessage`,
  `TranscriptionUpdateFrame`) are deprecated. Use `LLMUserAggregator`'s and
  `LLMAssistantAggregator`'s new events (`on_user_turn_stopped` and
  `on_assistant_turn_stopped`) instead.
  (PR [#3385](https://github.com/pipecat-ai/pipecat/pull/3385))

- Deprecated support for the `vad_events` `LiveOptions` in
  `DeepgramSTTService`. Instead, use a local Silero VAD for VAD events.
  Additionally, deprecated `should_interrupt` which will be removed along with
  `vad_events` support in a future release.
  (PR [#3386](https://github.com/pipecat-ai/pipecat/pull/3386))

- Loading external observers from files is deprecated, use the new pipeline
  task setup files and `PIPECAT_SETUP_FILES` environment variable instead.
  (PR [#3397](https://github.com/pipecat-ai/pipecat/pull/3397))

### Fixed

- Improved error handling in `ElevenLabsRealtimeSTTService`
  (PR [#3233](https://github.com/pipecat-ai/pipecat/pull/3233))

- Fixed an issue in `ElevenLabsRealtimeSTTService` causing an infinite loop
  that blocks the process if the websocket disconnects due to an error
  (PR [#3233](https://github.com/pipecat-ai/pipecat/pull/3233))

- Fixed a bug in `STTMuteFilter` where the user was not always muted during
  function calls, especially when there were multiple simultaneous calls.
  (PR [#3292](https://github.com/pipecat-ai/pipecat/pull/3292))

- Fixed a `RNNoiseFilter` issue that would cause a "[Errno 12] Cannot allocate
  memory" error when processing silence audio frames.
  (PR [#3322](https://github.com/pipecat-ai/pipecat/pull/3322))

- Updated `SpeechmaticsSTTService` for version `0.0.99+`:

  - Fixed `SpeechmaticsSTTService` to listen for `VADUserStoppedSpeakingFrame`
    in order to finalize transcription.
  - Default to `TurnDetectionMode.FIXED` for Pipecat-controlled end of turn
    detection.
  - Only emit VAD + interruption frames if VAD is enabled within the plugin
    (modes other than `TurnDetectionMode.FIXED` or `TurnDetectionMode.EXTERNAL`).
    (PR [#3328](https://github.com/pipecat-ai/pipecat/pull/3328))

- Fixed an issue with function calling where a handler failing to invoke its
  result callback could leave the context stuck in IN_PROGRESS, causing LLM
  inference for subsequent function call results to block while waiting on the
  unresolved call.
  (PR [#3343](https://github.com/pipecat-ai/pipecat/pull/3343))

- Fixed an issue with DeepgramTTSService where the model would output "Dot"
  instead of a period in some circumstances.
  (PR [#3345](https://github.com/pipecat-ai/pipecat/pull/3345))

- Fixed an issue in `traced_stt` where `model_name` in OpenTelemetry appears as
  `unknown`.
  (PR [#3351](https://github.com/pipecat-ai/pipecat/pull/3351))

- Fixed an issue in GeminiLiveLLMService where TranscriptionFrames were
  occasionally not pushed.
  (PR [#3356](https://github.com/pipecat-ai/pipecat/pull/3356))

- Fixed potential memory leaks and initialization issues in `KrispVivaFilter`
  by improving SDK lifecycle management.
  (PR [#3391](https://github.com/pipecat-ai/pipecat/pull/3391))

- Fixed timing issue in `BaseOutputTransport` where the bot speaking flag was
  set after awaiting, allowing the event loop to re-enter the method before the
  guard was set.
  (PR [#3400](https://github.com/pipecat-ai/pipecat/pull/3400))

- Fixed parallel function calling when using Gemini thinking.
  (PR [3420](https://github.com/pipecat-ai/pipecat/pull/3420))

- Fixed an issue in `traced_llm` where `model_name` in OpenTelemetry appears as
  `unknown`.
  (PR [#3422](https://github.com/pipecat-ai/pipecat/pull/3422))

- Fixed an issue in `traced_tts`, `traced_gemini_live`, and
  `traced_openai_realtime` where `model_name` in OpenTelemetry appears as
  `unknown`.
  (PR [#3428](https://github.com/pipecat-ai/pipecat/pull/3428))

- Fixed `request_image_frame` (for backwards compatibility) and restored
  function-call–related fields in `UserImageRequestFrame` and
  `UserImageRawFrame`, preventing a case where adding a non-LLM message to the
  context could trigger duplicate LLM inferences (on image arrival and on
  function-call result), potentially causing an infinite inference loop.
  (PR [#3430](https://github.com/pipecat-ai/pipecat/pull/3430))

- Fixed `LLMContext.create_audio_message()` by correcting an internal helper
  that was incorrectly declared async while being run in `asyncio.to_thread()`.
  (PR [#3435](https://github.com/pipecat-ai/pipecat/pull/3435))

### Other

- Added `52-live-transcription.py` foundational example demonstrating live
  transcription and translation from English to Spanish. In this example, the
  bot is not interruptible: as the user continues speaking, English
  transcriptions are queued, and the bot continuously translates and speaks
  each queued sentence in Spanish without being interrupted by new user speech.
  (PR [#3316](https://github.com/pipecat-ai/pipecat/pull/3316))

- Added a new foundational example `53-concurrent-llm-evaluation.py` that shows
  how to use `UserTurnProcessor`.
  (PR [#3372](https://github.com/pipecat-ai/pipecat/pull/3372))

- Added a new foundational example `28-user-assistant-turns.py` that shows how
  to use the new `LLMUserAggregator` and `LLMAssistantAggregator` events to
  gather a conversation transcript.
  (PR [#3385](https://github.com/pipecat-ai/pipecat/pull/3385))

## [0.0.98] - 2025-12-17

### Added

- Added `RimeNonJsonTTSService` which supports non-JSON streaming mode. This
  new class supports websocket streaming for the Arcana model.
  (PR [#3085](https://github.com/pipecat-ai/pipecat/pull/3085))

- Added additional functionality related to "thinking", for Google and
  Anthropic LLMs.

  1. New typed parameters for Google and Anthropic LLMs that control the
     models' thinking behavior (like how much thinking to do, and whether to
     output thoughts or thought summaries):
     - `AnthropicLLMService.ThinkingConfig`
     - `GoogleLLMService.ThinkingConfig`
  2. New frames for representing thoughts output by LLMs:
     - `LLMThoughtStartFrame`
     - `LLMThoughtTextFrame`
     - `LLMThoughtEndFrame`
  3. A generic mechanism for recording LLM thoughts to context, used
     specifically to support Anthropic, whose thought signatures are expected
     to appear alongside the text of the thoughts within assistant context
     messages. See:
     - `LLMThoughtEndFrame.signature`
     - `LLMAssistantAggregator` handling of the above field
     - `AnthropicLLMAdapter` handling of `"thought"` context messages
  4. Google-specific logic for inserting thought signatures into the context,
     to help maintain thinking continuity in a chain of LLM calls. See:
     - `GoogleLLMService` sending `LLMMessagesAppendFrame`s to add
       LLM-specific
       `"thought_signature"` messages to context
     - `GeminiLLMAdapter` handling of `"thought_signature"` messages
  5. An expansion of `TranscriptProcessor` to process LLM thoughts in
     addition to user and assistant utterances. See:
     - `TranscriptProcessor(process_thoughts=True)` (defaults to `False`)
     - `ThoughtTranscriptionMessage`, which is now also emitted with the
       `"on_transcript_update"` event
       (PR [#3175](https://github.com/pipecat-ai/pipecat/pull/3175))

- Data and control frames can now be marked as non-interruptible by using the
  `UninterruptibleFrame` mixin. Frames marked as `UninterruptibleFrame` will
  not be interrupted during processing, and any queued frames of this type will
  be retained in the internal queues. This is useful when you need ordered
  frames (data or control) that should not be discarded or cancelled due to
  interruptions.
  (PR [#3189](https://github.com/pipecat-ai/pipecat/pull/3189))

- Added `on_conversation_detected` event to `VoicemaiDetector`.
  (PR [#3207](https://github.com/pipecat-ai/pipecat/pull/3207))

- Added `x-goog-api-client` header with Pipecat's version to all Google
  services' requests.
  (PR [#3208](https://github.com/pipecat-ai/pipecat/pull/3208))

- Added support for the HeyGen LiveAvatar API (see https://www.liveavatar.com/).
  (PR [#3210](https://github.com/pipecat-ai/pipecat/pull/3210))

- Added to `AWSNovaSonicLLMService` functionality related to the new (and now
  default) Nova 2 Sonic model (`"amazon.nova-2-sonic-v1:0"`):

  - Added the `endpointing_sensitivity` parameter to control how quickly the
    model decides the user has stopped speaking.
  - Made the assistant-response-trigger hack a no-op. It's only needed for
    the older Nova Sonic model.
    (PR [#3212](https://github.com/pipecat-ai/pipecat/pull/3212))

- [Ultravox Realtime](https://docs.ultravox.ai) is now a supported
  speech-to-speech service.

  - Added `UltravoxRealtimeLLMService` for the integration.
  - Added `49-ultravox-realtime.py` example (with tool calling).
    (PR [#3227](https://github.com/pipecat-ai/pipecat/pull/3227))

- Added Daily PSTN dial-in support to the development runner with `--dialin`
  flag. This includes:

  - `/daily-dialin-webhook` endpoint that handles incoming Daily PSTN webhooks
  - Automatic Daily room creation with SIP configuration
  - `DialinSettings` and `DailyDialinRequest` types in `pipecat.runner.types`
    for type-safe dial-in data
  - The runner now mimics Pipecat Cloud's dial-in webhook handling for local
    development
    (PR [#3235](https://github.com/pipecat-ai/pipecat/pull/3235))

- Add Gladia session id to logs for `GladiaSTTService`.
  (PR [#3236](https://github.com/pipecat-ai/pipecat/pull/3236))

- Added `InworldHttpTTSService` which uses Inworld's HTTP based TTS service in
  either streaming or non-streaming mode. Note: This class was previously named
  `InworldTTSService`.
  (PR [#3239](https://github.com/pipecat-ai/pipecat/pull/3239))

- Added `language_hints_strict` parameter to `SonioxSTTService` to strictly
  enforces language hints. This ensures that transcription occurs in the
  specified language.
  (PR [#3245](https://github.com/pipecat-ai/pipecat/pull/3245))

- Added Pipecat library version info to the `about` field in the `bot-ready`
  RTVI message.
  (PR [#3248](https://github.com/pipecat-ai/pipecat/pull/3248))

- Added `VisionFullResponseStartFrame`, `VisionFullResponseEndFrame` and
  `VisionTextFrame`. This are used by vision services similar to LLM
  services.
  (PR [#3252](https://github.com/pipecat-ai/pipecat/pull/3252))

### Changed

- `FunctionCallInProgressFrame` and `FunctionCallResultFrame` have changed from
  system frames to a control frame and a data frame, respectively, and are
  now both marked as `UninterruptibleFrame`.
  (PR [#3189](https://github.com/pipecat-ai/pipecat/pull/3189))

- `UserBotLatencyLogObserver` now uses `VADUserStartedSpeakingFrame` and
  `VADUserStoppedSpeakingFrame` to determine latency from user stopped speaking
  to bot started speaking.
  (PR [#3206](https://github.com/pipecat-ai/pipecat/pull/3206))

- Updated `HeyGenVideoService` and `HeyGenTransport` to support both HeyGen
  APIs (Interactive Avatar and Live Avatar).
  Using them is as simple as specifying the `service_type` when creating the
  `HeyGenVideoService` and the `HeyGenTransport`:

  ```python
  heyGen = HeyGenVideoService(
      api_key=os.getenv("HEYGEN_LIVE_AVATAR_API_KEY"),
      service_type=ServiceType.LIVE_AVATAR,
      session=session,
  )
  ```

  (PR [#3210](https://github.com/pipecat-ai/pipecat/pull/3210))

- Made `"amazon.nova-2-sonic-v1:0"` the new default model for
  `AWSNovaSonicLLMService`.
  (PR [#3212](https://github.com/pipecat-ai/pipecat/pull/3212))

- Updated the `run_inference` methods in the LLM service classes
  (`AnthropicLLMService`, `AWSBedrockLLMService`, `GoogleLLMService`, and
  `OpenAILLMService` and its base classes) to use the provided LLM
  configuration parameters.
  (PR [#3214](https://github.com/pipecat-ai/pipecat/pull/3214))

- Updated default models for:

  - `GeminiLiveLLMService` to `gemini-2.5-flash-native-audio-preview-12-2025`.
  - `GeminiLiveVertexLLMService` to `gemini-live-2.5-flash-native-audio`.
    (PR [#3228](https://github.com/pipecat-ai/pipecat/pull/3228))

- Changed the `reason` field in `EndFrame`, `CancelFrame`, `EndTaskFrame`, and
  `CancelTaskFrame` from `str` to `Any` to indicate that it can hold values
  other than strings.
  (PR [#3231](https://github.com/pipecat-ai/pipecat/pull/3231))

- Updated websocket STT services to use the `WebsocketSTTService` base class.
  This base class manages the websocket connection and handles reconnects.
  Updated services:

  - `AssemblyAISTTService`
  - `AWSTranscribeSTTService`
  - `GladiaSTTService`
  - `SonioxSTTService`
    (PR [#3236](https://github.com/pipecat-ai/pipecat/pull/3236))

- Changed Inworld's TTS service implementations:

  - Previously, the HTTP implementation was named `InworldTTSService`. That
    has been moved to `InworldHttpTTSService`. This service now supports
    word-timestamp alignment data in both streaming and non-streaming modes.
  - Updated the `InworldTTSService` class to use Inworld's Websocket API.
    This class now has support for word-timestamp alignment data and tracks
    contexts for each user turn.
    (PR [#3239](https://github.com/pipecat-ai/pipecat/pull/3239))

- ⚠️ Breaking change: `WordTTSService.start_word_timestamps()` and
  `WordTTSService.reset_word_timestamps()` are now async.
  (PR [#3240](https://github.com/pipecat-ai/pipecat/pull/3240))

- Updated the current RTVI version to 1.1.0 to reflect recent additions and
  deprecations.

  - New RTVI Messages: `send-text` and `bot-output`
  - Deprecated Messages: `append-to-context` and `bot-transcription`
    (PR [#3248](https://github.com/pipecat-ai/pipecat/pull/3248))

- `MoondreamService` now pushes `VisionFullResponseStartFrame`,
  `VisionFullResponseEndFrame` and `VisionTextFrame`.
  (PR [#3252](https://github.com/pipecat-ai/pipecat/pull/3252))

### Deprecated

- `FalSmartTurnAnalyzer` and `LocalSmartTurnAnalyzer` are deprecated and will
  be removed in a future version. Use `LocalSmartTurnAnalyzerV3` instead.
  (PR [#3219](https://github.com/pipecat-ai/pipecat/pull/3219))

### Removed

- Removed the deprecated VLLM-based open source Ultravox STT service.
  (PR [#3227](https://github.com/pipecat-ai/pipecat/pull/3227))

### Fixed

- Fixed a bug in `AWSNovaSonicLLMService` where we would mishandle cancelled
  tool calls in the context, resulting in errors.
  (PR [#3212](https://github.com/pipecat-ai/pipecat/pull/3212))

- Better support conversation history with Gemini 2.5 Flash Image (model
  "gemini-2.5-flash-image"). Prior to this fix, the model had no memory of
  previous images it had generated, so it wouldn't be able to iterate on
  them.
  (PR [#3224](https://github.com/pipecat-ai/pipecat/pull/3224))

- Support conversations with Gemini 3 Pro Image (model
  "gemini-3-pro-image-preview"). Prior to this fix, after the model generated
  an image the conversation would not be able to progress.
  (PR [#3224](https://github.com/pipecat-ai/pipecat/pull/3224))

- Fixed an issue where `ElevenLabsHttpTTSService` was not updating
  voice settings when receiving a `TTSUpdateSettingsFrame`.
  (PR [#3226](https://github.com/pipecat-ai/pipecat/pull/3226))

- Fixed the return type for `SmallWebRTCRequestHandler.handle_web_request()`
  function.
  (PR [#3230](https://github.com/pipecat-ai/pipecat/pull/3230))

- Fix a bug in LLM context audio content handling
  (PR [#3234](https://github.com/pipecat-ai/pipecat/pull/3234))

- In `GladiaSTTService`, reset the `_bytes_sent` counter on connecting the
  websocket. This avoids unnecessary audio buffer trimming.
  (PR [#3236](https://github.com/pipecat-ai/pipecat/pull/3236))

- Fixed a TTS service word-timestamp issue that could cause generated
  `TTSTextFrame` instances to have an incorrect pts (`pts = -1`).
  (PR [#3240](https://github.com/pipecat-ai/pipecat/pull/3240))

- Fixed an issue in `SimpleTextAggreagtor` where spaces were not being stripped
  before returning the aggregation. This resulted in an extra space for TTS
  services that don't support word-timestamp alignment data.
  (PR [#3247](https://github.com/pipecat-ai/pipecat/pull/3247))
