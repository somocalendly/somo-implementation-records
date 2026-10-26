---
name: SCH-4682 description dark mode
overview: The Quill description field is tight under its label and uses the wrong text color in dark mode. Match CUI label spacing and dark-mode tokens.
todos:
  - id: dark-tokens
    content: Set the description editor text and border to CUI dark-mode tokens
    status: pending
  - id: label-gap
    content: Match the label gap used by other editor fields, unless SCH-4911 already did
    status: pending
  - id: dark-test
    content: Assert dark-mode text and border tokens on the description editor
    status: pending
isProject: false
branch: SCH-4682-description-spacing
pullRequest: https://github.com/calendly/mf-publisher/pull/4115
started: 2026-10-22
pullRequestOpened: 2026-10-22
released: 2026-10-23
delivery:
  - repo: calendly/mf-publisher
    branch: SCH-4682-description-spacing
    pullRequest: https://github.com/calendly/mf-publisher/pull/4115
---

# Fix description spacing and dark mode

Shareable implementation note for [SCH-4682](https://calendlyapp.atlassian.net/browse/SCH-4682) (ET Scheduling · 3 SP · Ready to Refine · P4 · Low impact).

SLA Breach Date is not set. Parent: [SCH-4200](https://calendlyapp.atlassian.net/browse/SCH-4200).

Repo: `calendly/mf-publisher`. Local checkout: `/Users/somo.banerjee/Documents/projects/mf-publisher`.

This ticket is Ready to Refine. The acceptance criteria are already on the ticket. Confirm them before coding if refine changes the copy.

## Goal

Two things on the Meeting details description field:

- The gap between the “Description” label and the editor matches the other CUI fields.
- In dark mode the text is light, and the border uses CUI tokens.

The label is separate because the editor hides its own label:

```61:68:app/src/javascripts/features/event_type_editor_crud/tabs/meeting_details/meeting_details_tab.js
      <div className={descriptionSectionCss}>
        <LabelText as="label" htmlFor="wysiwyg-description" size="large">
          {I18n.t('publisher.event_type_editor_crud.meeting_details.description_label')}
        </LabelText>
        <Description
          eventTypeId={eventType.id}
          isLocked={isPanelLocked('description')}
          hideLabel
        />
```

`Description` is the Quill field from `components/event_type_edit`. Quill does not follow `TextField` dark-mode colors. [SCH-4911](https://calendlyapp.atlassian.net/browse/SCH-4911) is the default-state outline and the same label gap. If that change has landed, this ticket is the dark-mode text and border only.

## Out of scope

- Quill toolbar behavior and the description HTML that gets saved.
- A second copy of the label-gap change if SCH-4911 already added it.

## Code changes

1. In dark mode, set the editor text to the CUI text token (`colorTextStandard` resolves for the active theme) and the border to `colorStrokeStandard`. Scope the override to this editor.
2. If the label gap is still tighter than Meeting name, add the same gap on `descriptionSectionCss`.
3. Keep `htmlFor="wysiwyg-description"`.

## Tests

From `app/`:

`yarn test src/javascripts/features/event_type_editor_crud/tabs/meeting_details/__tests__/meeting_details_tab.test.js`

Assert the editor’s text color and border use those tokens under the dark theme. Assert the label gap if this change still owns it.

## Manual check

1. Open the new editor, Meeting details, in light mode. The label gap matches Meeting name.
2. Switch to dark mode. Description text is light and the border matches the other inputs.
3. Typed text stays light, including a second paragraph.

## Sources

- [SCH-4682](https://calendlyapp.atlassian.net/browse/SCH-4682)
