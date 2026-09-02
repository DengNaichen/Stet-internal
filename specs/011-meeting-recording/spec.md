# Feature Specification: Person-to-Person Meeting Recording

**Feature Branch**: `011-meeting-recording`
**Created**: 2026-09-03
**Status**: Draft
**Input**: Replace always-on passive listening as the product path with an explicit person-to-person meeting recording mode. A dedicated, hard-to-press hotkey toggles recording. Stop writes a timestamped meeting folder with audio and a speaker-labeled transcript. No rewrite, paste, or LLM. A Meetings sidebar explains the shortcut, where files live, and what they look like.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Toggle a face-to-face meeting (Priority: P1)

As a Mac user, I press a dedicated meeting shortcut to start recording people in the room. I press it again to stop. Stet saves a folder named with the start time containing the audio and a transcript labeled by speaker. Dictation and rewrite are not involved.

**Why this priority**: This is the whole product slice. Always-on passive capture is not required for this story.

**Independent Test**: Press the meeting shortcut, speak as two people, press it again, open the Meetings sidebar, reveal the folder, and confirm `audio.wav` plus `transcript.md` exist with timestamps and speaker labels.

**Acceptance Scenarios**:

1. **Given** the app is running with microphone permission and dictation is idle, **When** the user presses the meeting shortcut, **Then** meeting recording starts and a visible recording status is shown.
2. **Given** a meeting is recording, **When** the user presses the meeting shortcut again, **Then** recording stops, the session is processed, and a meeting folder is written.
3. **Given** a completed meeting folder, **When** the user opens it, **Then** it contains the captured audio, a markdown transcript with speaker labels and start time, and no rewritten or pasted text.

---

### User Story 2 - Find the shortcut and the files (Priority: P2)

As a user, I can open a Meetings sidebar page that shows how to change the shortcut, where meeting folders are stored, and an example of the file layout.

**Why this priority**: The shortcut is intentionally uncommon; the sidebar is how people discover it.

**Independent Test**: Open Settings → Meetings, change the shortcut, click Reveal Meetings Folder, and read the example layout.

**Acceptance Scenarios**:

1. **Given** Settings is open, **When** the user selects Meetings, **Then** they can edit the meeting shortcut, reveal the meetings directory, and see what a meeting folder contains.
2. **Given** the default shortcut has never been changed, **When** the user inspects it, **Then** it is Control-Option-Command-M.

---

### User Story 3 - Stay out of dictation (Priority: P3)

As a user, meeting recording and dictation do not run at the same time, and a meeting never goes through refine or auto-paste.

**Why this priority**: Prevents microphone fights and keeps meeting notes faithful.

**Independent Test**: Start a meeting, press the dictation hotkey, confirm dictation does not start; finish the meeting and confirm History/refine were not invoked.

**Acceptance Scenarios**:

1. **Given** a meeting is recording or processing, **When** the user presses the dictation hotkey, **Then** dictation does not start.
2. **Given** dictation is listening, **When** the user presses the meeting shortcut, **Then** meeting recording does not start.
3. **Given** a meeting finishes, **When** the transcript is written, **Then** no rewrite request and no paste/copy into another app occur.

---

### Edge Cases

- Microphone permission denied: recording does not start; the failure is visible.
- Empty or near-silent meeting: audio is still saved; transcript may note that no speech was recognized.
- More than four speakers: at most four speaker labels are used; additional people may share a label.
- Overlapping speech: that interval is labeled unresolved rather than split into separate audio streams.
- Capture failure mid-meeting: already captured audio is kept in the folder with a failure note in the transcript.
- Passive listening may be on: starting a meeting suspends it; stopping a meeting restores it when it is still enabled.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: Mac MUST provide a dedicated meeting shortcut, defaulting to Control-Option-Command-M, that toggles meeting recording on key-down.
- **FR-002**: The user MUST be able to change that shortcut from the Meetings sidebar.
- **FR-003**: Starting a meeting MUST capture microphone audio until the user stops it, without waiting for an enrolled owner voice.
- **FR-004**: Stopping a meeting MUST write a folder named with the local start timestamp under the app’s Meetings directory.
- **FR-005**: That folder MUST contain the session audio and a markdown transcript with start time, duration, and speaker-labeled turns.
- **FR-006**: Speaker labels MUST distinguish Me when an enrolled owner profile matches a track, otherwise Speaker 1–4, and Unresolved for overlapping or unclassifiable speech.
- **FR-007**: Meeting output MUST NOT invoke rewrite, auto-paste, or dictation History delivery.
- **FR-008**: Meeting recording and dictation MUST be mutually exclusive.
- **FR-009**: The Meetings sidebar MUST explain the shortcut, reveal the meetings folder, and show the file layout.
- **FR-010**: A meeting in progress MUST be visibly indicated in the menu bar.
- **FR-011**: Starting a meeting MUST suspend passive listening for the session; stopping MUST restore passive listening when it remains enabled.

### Scope Boundaries

- Person-to-person / in-room microphone capture only. System audio, Zoom, and other meeting-app loopback are out of scope.
- Source separation of overlapping talkers is out of scope.
- Automatic naming of unknown people is out of scope.
- Summaries, action items, and LLM cleanup are out of scope.
- Always-on passive admission (pending buffer, relevance deadline, owner gate) is not part of this feature.
- iPhone meeting recording is out of scope.

### Key Entities

- **Meeting Session**: One explicit recording interval with start and end time.
- **Meeting Folder**: Timestamp-named directory holding `audio.wav`, `transcript.md`, and `session.json`.
- **Speaker Turn**: An ordered transcript region labeled Me, Speaker N, a known enrolled name, or Unresolved.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: In a two-person in-room test, stopping the meeting produces a folder that contains audio and a transcript with at least two distinct speaker labels when both voices were present.
- **SC-002**: 100% of completed meetings skip rewrite and paste.
- **SC-003**: 100% of attempts to start dictation during an active meeting leave dictation idle.
- **SC-004**: A user who has never opened Meetings can change the shortcut and reveal the meetings folder from that sidebar alone.

## Assumptions

- Microphone permission is granted before relying on the shortcut.
- Consent and local recording-law compliance remain the user’s responsibility.
- Sortformer’s four-speaker ceiling is accepted for this version.
- Processing after stop may take time proportional to meeting length.
