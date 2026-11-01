# QRiosity: project assessment and repurposing options

Discussion notes recorded on November 1, 2026.

## Purpose of these notes

Capture our initial repository assessment, the questions discussed, and the
available directions for the project. These are planning notes, not a decision
to implement a particular direction. No application changes have been made as
part of this assessment.

## What the project is

QRiosity is an AI-assisted learning platform intended to help people choose
learning resources, follow roadmaps, and maintain a practice habit. Its apparent
target problem is: “I want to learn something, but I don't know where to start,
which resources to choose, or how to stay consistent.”

The intended experience is to:

1. Create an account and select interests.
2. Browse community roadmaps or request an AI-generated roadmap for a topic.
3. Follow the linked courses and resources and track roadmap progress.
4. Answer daily questions based on selected interests.
5. Receive AI ratings and feedback on those answers.

Users can also contribute course entries and assemble roadmaps. Course records
primarily describe external resources through URLs and metadata; this is not
evidence of a complete course-hosting system.

## Evidence and architecture

The assessment used Repomix to map and compress the repository, followed by
direct inspection of the README, schema, application components, Lambda
handlers, AI functions, and deployment workflows.

| Area | Implementation observed |
| --- | --- |
| Frontend | React, TypeScript, Vite, PrimeReact |
| API | GraphQL through AWS AppSync; Apollo client in the frontend |
| Backend | Go AWS Lambda handlers for authentication, learning, and challenges |
| Data | Active handlers use DynamoDB repositories |
| Authentication | Amazon Cognito |
| AI | Python Lambda functions using the OpenAI SDK |
| Delivery | GitHub Actions; frontend deployment to S3 and CloudFront |

Older MongoDB repositories and a local LM Studio integration also remain in the
codebase. Their presence does not mean they are active production dependencies.

Useful source references:

- [Project overview](../Readme.md)
- [GraphQL schema](../src/backend/schema/schema.graphql)
- [Learning backend](../src/backend/cmd/learning/main.go)
- [Daily challenge backend](../src/backend/cmd/daily/main.go)
- [AI roadmap generator](../src/ai/get-roadmap/lambda_function.py)
- [Pricing page](../src/frontend/src/misc/Pricing.tsx)
- [Usage limits](../src/backend/internal/constants/constants.go)
- [Deployment workflows](../.github/workflows/)

The application has not been run during this assessment. Cloud service
availability, deployment status, and end-to-end behavior have not been verified.

## Was it intended as a SaaS?

The evidence suggests a subscription SaaS design. The pricing page lists:

| Plan | Price written in the repository | Intended differentiation |
| --- | --- | --- |
| Student | Free | Limited roadmap access, daily practice, and progress tracking |
| Apprentice | $3.99/month | Higher limits, roadmap creation, and AI generation tickets |
| Creator | $8.99/month | Additional generation and advertised creator benefits |

These are repository values, not verified current commercial offers. The plan
selection buttons have no purchase handlers in the inspected component, so the
pricing page does not establish an implemented checkout or billing system.
Advertised benefits should not be assumed to be fully implemented or consistent
with every backend limit.

Our assessment was an early SaaS/MVP with substantial code, rather than a proven
commercial product. The repository does not establish whether it launched,
attracted customers, or validated demand.

## OpenAI key and model choice

All three Python AI functions currently hardcode `gpt-4o-mini`:

- `ask_question`: generate practice questions.
- `rate_question`: rate answers and return feedback.
- `get-roadmap`: generate a structured roadmap with courses and resource links.

The AI functions expect an `OPENAI_API_KEY` environment variable. It belongs in
the backend/Lambda environment, not in browser code or committed configuration.
An API key alone will not make the full application work: the existing AWS
authentication, API, storage, and service configuration are also required.

We considered cost effectiveness a plausible reason for the original model
choice, but found no documented statement of the programmer's reasoning.

The user's preferred replacement is GPT-6 Luna, whose API model ID was verified
as `gpt-6-luna` in the official OpenAI documentation during the discussion:
[GPT-6 Luna model documentation](https://developers.openai.com/api/docs/models/gpt-6-luna).
Availability and compatibility should be checked again when implementing a
migration; no pricing comparison or application-specific quality evaluation was
performed.

A proposed migration would:

1. Make the model configurable through `OPENAI_MODEL`, with the chosen default.
2. Update the old OpenAI Python SDK dependency as needed.
3. Check request parameters and structured-output compatibility.
4. Validate roadmap JSON, question quality, and grading behavior on representative
   examples before switching deployed traffic.

No model migration, key configuration, or paid API calls have been performed.

## Was the code created with AI assistance?

Our tentative answer was “probably,” with moderate confidence, not a finding of
authorship. Observations included placeholder comments beside configured values,
comments explaining routine operations, leftover template documentation and
integrations, and features with uneven completeness.

These patterns also occur in manually written prototypes and tutorial-based
projects. Using OpenAI as a product dependency is not evidence that AI wrote the
code. Commit history could provide additional context, but was not analyzed for
authorship. The hypothesis should not be treated as a confirmed fact.

## What a chatbot can already do

A chatbot can generate and adapt roadmaps, explain concepts, suggest resources,
ask practice questions, and provide feedback. Roadmap generation by itself is
therefore a weak differentiator for a standalone subscription product.

A dedicated application could add durable organization and follow-through:

| Chatbot capability | Potential application value |
| --- | --- |
| Suggest a learning path | Maintain a searchable library of reviewed roadmaps |
| Recommend the next topic | Use recorded progress to show a clear next action |
| Ask questions and give feedback | Retain practice history and show improvement |
| Suggest resources | Maintain links that have been checked and curated |

These are opportunities, not claims that all of this behavior currently exists.

## Available directions

The user already has created and saved multiple roadmaps. Their location,
format, structure, and import requirements have not yet been inspected.

| Option | What it would provide | Main work or tradeoff |
| --- | --- | --- |
| Use a chatbot with the saved roadmaps | Explanations, adaptations, practice, and feedback with minimal development | Organization and progress would need an explicit workflow |
| Repurpose QRiosity as a personal learning dashboard | A library of the user's roadmaps, linked resources, completion tracking, and clear next steps | Inspect/import existing roadmaps and simplify the product for personal use |
| Add a roadmap-aware AI tutor | Guidance informed by the selected roadmap and recorded progress | Pass relevant context to the model and validate feedback quality |
| Develop a focused learning SaaS | Reviewed paths and follow-through for a defined audience, such as certification or career-transition learners | Validate demand, maintain resources, and finish billing and operational requirements |
| Continue the original broad SaaS | General roadmap discovery, generation, community creation, and daily challenges | Broad scope; roadmap generation alone provides limited differentiation from chatbots |

These options can be combined or pursued in stages. The architecture and hosting
choice remain open; keeping AWS versus simplifying deployment has not been
evaluated.

## Suggested starting direction

Our recommendation was to start with a personal learning dashboard built around
the user's existing roadmaps, then add a tutor that understands the selected
roadmap and progress. This would provide immediate personal value and help
establish whether there is a useful product to offer to others.

If pursuing a business later, a defined audience and reviewed, maintained
learning paths would provide a clearer value proposition than generic AI
generation alone.

## Open questions and next steps

1. Locate and inspect the saved roadmaps and map their structure to the current
   course/roadmap model.
2. Choose the initial audience: personal use, a small group, or paying customers.
3. Identify the smallest useful workflow: import, browse, follow links, record
   completion, and see the next step.
4. Verify what currently builds and runs and which AWS resources are available.
5. Decide whether to retain the current cloud architecture or simplify it.
6. Implement and evaluate the preferred model migration if AI features are kept.
7. Add commercial features only if the SaaS direction is selected.

No repurposing direction, infrastructure change, or implementation scope has
been finalized.
