# kr-job-helper (국내채용도우미)

A Claude Code skill that sweeps Korean job sites in one go, matches postings against your own résumé and criteria, and emails you a short report with links and a "why this fits you" note for each pick.

한국 채용사이트(사람인·잡코리아·인크루트·원티드·캐치·링크드인·워크넷 등)를 한 번에 훑어 내 이력에 맞는 공고를 고르고, 추천 이유와 링크가 든 보고서를 메일로 보내 주는 Claude Code 스킬입니다.

## What it does
1. Reads your profile (your logged-in Saramin résumé, or `profile/` files you provide).
2. Searches 7+ Korean job sites with your keywords (browser automation via ego-browser).
3. Drops postings you've already seen, then reads details and checks them against your criteria.
4. Emails the top picks to yourself, each with a reason it fits and its risks.

It never submits applications, creates accounts, or bypasses logins/bot checks.

## Install
Copy `SKILL.md` to `~/.claude/skills/kr-job-helper/SKILL.md`, then ask Claude Code: "채용공고 찾아줘" or `/kr-job-helper`.

Requires: Claude Code, the `ego-browser` skill (or another browser tool), and a Gmail connector if you want the email step.

## Notes
- Site URLs and extraction tips were verified in Nov 2026; sites change layouts, so adjust as needed.
- Your résumé and personal details stay on your machine; this repo contains none.

## License
Personal, non-commercial use only. Commercial use, resale, and redistribution require written permission. See `LICENSE`. (개인 취업용은 무료, 상업적 이용은 허락 필요)
