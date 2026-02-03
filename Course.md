Yep — you can scaffold this with (1) JSON “source of truth”, (2) Markdown content files, and (3) a Prisma schema that can ingest either approach (JSON-driven seeding + MD-driven rendering).

Below is a solid, extensible baseline you can copy into your repo and start wiring immediately.

⸻

Suggested repo layout

/content
  /courses
    /email-essentials
      course.json
      syllabus.md
      /modules
        01-getting-started.md
        02-writing-email.md
        03-safety-and-scams.md
/prisma
  schema.prisma
/scripts
  seed-content.ts

	•	course.json = canonical structure + IDs/slugs
	•	syllabus.md = marketing + overview + outcomes
	•	modules/*.md = lesson content (frontmatter + body)

⸻

1) Course JSON scaffolding (per course)

/content/courses/email-essentials/course.json

{
  "version": "1.0",
  "course": {
    "slug": "email-essentials",
    "title": "Email Essentials",
    "status": "draft",
    "category": "internet",
    "level": "beginner",
    "locale": "en-US",
    "coverImage": "cover.png",
    "shortDescription": "Master email communication — setup, writing, organization, and scam avoidance.",
    "fullDescription": "Learn the fundamentals of email: accounts, composition, attachments, inbox workflows, and security.",
    "estimatedMinutes": 60,
    "price": {
      "currency": "USD",
      "amountCents": 2900
    },
    "tags": ["email", "communication", "security"],
    "learningOutcomes": [
      "Set up and manage email accounts",
      "Write clear, professional emails",
      "Organize an inbox effectively",
      "Recognize and avoid phishing and scams"
    ],
    "requirements": {
      "prerequisites": [
        {
          "type": "course",
          "slug": "internet-basics",
          "required": false,
          "note": "Helpful but not required."
        }
      ],
      "recommended": [
        {
          "type": "course",
          "slug": "passwords-and-2fa",
          "note": "Pairs well with scam avoidance."
        }
      ]
    },
    "certification": {
      "enabled": true,
      "certificateTemplate": "default"
    },
    "publishing": {
      "drip": {
        "enabled": false,
        "daysBetweenModules": 0
      }
    }
  },
  "curriculum": {
    "modules": [
      {
        "slug": "getting-started",
        "title": "Getting Started",
        "order": 1,
        "estimatedMinutes": 15,
        "lessons": [
          {
            "slug": "what-is-email",
            "title": "What Email Is (and Isn’t)",
            "order": 1,
            "type": "lesson",
            "contentFile": "modules/01-getting-started.md#what-is-email",
            "estimatedMinutes": 5,
            "assessment": null
          },
          {
            "slug": "create-an-account",
            "title": "Creating an Email Account",
            "order": 2,
            "type": "lesson",
            "contentFile": "modules/01-getting-started.md#create-an-account",
            "estimatedMinutes": 10,
            "assessment": {
              "type": "quiz",
              "passingScore": 70
            }
          }
        ]
      },
      {
        "slug": "writing-email",
        "title": "Writing Email That Works",
        "order": 2,
        "estimatedMinutes": 20,
        "lessons": [
          {
            "slug": "subject-lines",
            "title": "Subject Lines & Structure",
            "order": 1,
            "type": "lesson",
            "contentFile": "modules/02-writing-email.md#subject-lines",
            "estimatedMinutes": 10,
            "assessment": null
          },
          {
            "slug": "tone-and-clarity",
            "title": "Tone, Clarity, and Etiquette",
            "order": 2,
            "type": "lesson",
            "contentFile": "modules/02-writing-email.md#tone-and-clarity",
            "estimatedMinutes": 10,
            "assessment": {
              "type": "assignment",
              "rubric": "basic-writing"
            }
          }
        ]
      },
      {
        "slug": "safety",
        "title": "Safety & Scam Avoidance",
        "order": 3,
        "estimatedMinutes": 25,
        "lessons": [
          {
            "slug": "phishing-basics",
            "title": "Phishing 101",
            "order": 1,
            "type": "lesson",
            "contentFile": "modules/03-safety-and-scams.md#phishing-basics",
            "estimatedMinutes": 10,
            "assessment": null
          },
          {
            "slug": "spotting-red-flags",
            "title": "Red Flags & Verification",
            "order": 2,
            "type": "lesson",
            "contentFile": "modules/03-safety-and-scams.md#spotting-red-flags",
            "estimatedMinutes": 15,
            "assessment": {
              "type": "quiz",
              "passingScore": 80
            }
          }
        ]
      }
    ],
    "completion": {
      "rule": "all_required_lessons",
      "requiredLessonTypes": ["lesson", "quiz", "assignment"]
    }
  }
}

This supports:
	•	per-course curriculum
	•	optional + required prereqs
	•	linking MD anchors for content rendering
	•	quizzes/assignments without boxing you in

⸻

2) Markdown scaffolding

syllabus.md (per course)

/content/courses/email-essentials/syllabus.md

---
slug: email-essentials
title: Email Essentials
category: internet
level: beginner
estimatedMinutes: 60
---

# Email Essentials

## What you’ll learn
- How email works at a practical level
- Writing effective subject lines and messages
- Attachments, replies, and organization workflows
- Spotting scams and phishing attempts

## Who this is for
People who want to be confident using email personally or professionally.

## Completion
Finish all required lessons and pass the quizzes to earn your certificate.

Module content with anchors

/content/courses/email-essentials/modules/01-getting-started.md

---
moduleSlug: getting-started
moduleTitle: Getting Started
order: 1
---

# Getting Started

## What Email Is (and Isn’t) {#what-is-email}
Explain inbox/outbox, providers, addresses, clients vs webmail.

## Creating an Email Account {#create-an-account}
Step-by-step outline. Include tips like recovery email, strong passwords, 2FA.

Same pattern for other module files.

⸻

3) Prisma schema (multi-course platform with prerequisites)

Here’s a practical schema that supports:
	•	courses
	•	modules/lessons
	•	course prerequisites (required vs recommended)
	•	programs/tracks (optional, but useful for “required courses for some”)
	•	enrollments + progress
	•	assessments scaffolding

/prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

enum CourseStatus {
  DRAFT
  PUBLISHED
  ARCHIVED
}

enum CourseLevel {
  BEGINNER
  INTERMEDIATE
  ADVANCED
}

enum LessonType {
  LESSON
  QUIZ
  ASSIGNMENT
}

enum PrereqType {
  REQUIRED
  RECOMMENDED
}

model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String?
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  enrollments Enrollment[]
  progress    LessonProgress[]
}

model Course {
  id               String       @id @default(cuid())
  slug             String       @unique
  title            String
  shortDescription String
  fullDescription  String
  status           CourseStatus @default(DRAFT)
  level            CourseLevel
  category         String
  locale           String       @default("en-US")
  coverImage       String?
  estimatedMinutes Int?
  priceCents       Int?
  currency         String       @default("USD")

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  modules        Module[]
  enrollments    Enrollment[]
  prerequisites  CoursePrerequisite[] @relation("course_prereqs_course")
  requiredFor    CoursePrerequisite[] @relation("course_prereqs_prereq")

  @@index([status, category])
}

model CoursePrerequisite {
  id             String     @id @default(cuid())
  courseId       String
  prerequisiteId String
  type           PrereqType @default(REQUIRED)
  note           String?

  course       Course @relation("course_prereqs_course", fields: [courseId], references: [id], onDelete: Cascade)
  prerequisite Course @relation("course_prereqs_prereq", fields: [prerequisiteId], references: [id], onDelete: Cascade)

  @@unique([courseId, prerequisiteId])
  @@index([courseId, type])
}

model Module {
  id               String @id @default(cuid())
  courseId         String
  slug             String
  title            String
  order            Int
  estimatedMinutes Int?

  course  Course   @relation(fields: [courseId], references: [id], onDelete: Cascade)
  lessons Lesson[]

  @@unique([courseId, slug])
  @@unique([courseId, order])
  @@index([courseId, order])
}

model Lesson {
  id               String    @id @default(cuid())
  moduleId         String
  slug             String
  title            String
  order            Int
  type             LessonType @default(LESSON)
  contentPath      String?   // e.g. "modules/01-getting-started.md#what-is-email"
  estimatedMinutes Int?

  module   Module          @relation(fields: [moduleId], references: [id], onDelete: Cascade)
  progress LessonProgress[]
  assessment Assessment?

  @@unique([moduleId, slug])
  @@unique([moduleId, order])
  @@index([moduleId, order])
}

model Assessment {
  id         String  @id @default(cuid())
  lessonId   String  @unique
  kind       String  // "quiz" | "assignment" etc (string for flexibility)
  configJson Json    // stores rubric, passingScore, question refs, etc

  lesson Lesson @relation(fields: [lessonId], references: [id], onDelete: Cascade)
}

model Enrollment {
  id        String   @id @default(cuid())
  userId    String
  courseId  String
  createdAt DateTime @default(now())

  user   User   @relation(fields: [userId], references: [id], onDelete: Cascade)
  course Course @relation(fields: [courseId], references: [id], onDelete: Cascade)

  @@unique([userId, courseId])
  @@index([courseId])
}

model LessonProgress {
  id        String   @id @default(cuid())
  userId    String
  lessonId  String
  status    String   @default("NOT_STARTED") // NOT_STARTED | IN_PROGRESS | COMPLETED
  score     Int?
  updatedAt DateTime @updatedAt
  createdAt DateTime @default(now())

  user   User   @relation(fields: [userId], references: [id], onDelete: Cascade)
  lesson Lesson @relation(fields: [lessonId], references: [id], onDelete: Cascade)

  @@unique([userId, lessonId])
  @@index([userId, status])
}

model Program {
  id          String   @id @default(cuid())
  slug        String   @unique
  title       String
  description String?
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  requirements ProgramCourseRequirement[]
}

model ProgramCourseRequirement {
  id        String @id @default(cuid())
  programId String
  courseId  String
  required  Boolean @default(true)
  order     Int?

  program Program @relation(fields: [programId], references: [id], onDelete: Cascade)
  course  Course  @relation(fields: [courseId], references: [id], onDelete: Cascade)

  @@unique([programId, courseId])
  @@index([programId, required])
}

Why Program matters: it cleanly models “required courses for some” (some users enroll in a program/track that mandates certain courses, while standalone course buyers don’t).

⸻

4) Wiring strategy (JSON + MD → DB)

The typical flow:
	•	Parse /content/**/course.json
	•	Upsert Course by slug
	•	Upsert Modules/Lessons by (courseId, slug) and (moduleId, slug)
	•	Store contentPath pointing at MD anchors
	•	Render MD at runtime (or pre-render) for lessons

⸻

Implementation checklist
	•	Create /content/courses/<slug>/course.json + syllabus.md + modules/*.md
	•	Add Prisma schema and run initial migration
	•	Write seed script to upsert courses/modules/lessons from JSON
	•	Add prerequisite linking (CoursePrerequisite) during seed
	•	Add program/track seeding (optional)
	•	Build API: list courses, course detail (modules + lessons), enroll, progress updates
	•	Build renderer: contentPath → load MD → scroll to anchor (or split at build time)

⸻

If you want, I can also give you a seed-content.ts template (Node/TS + Prisma Client) that ingests that exact JSON shape and upserts everything deterministically.
