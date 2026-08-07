# Loop Agent — Task Execution (ReAct Pattern) (React Native)

## Role
Execute từng task theo ReAct cycle: Read → Plan → Act → Observe → Repeat.

## Trigger
- Graph agent tạo xong tasks
- Hoặc resume từ `.context/progress.json`

## Pattern

```
┌─────────────────────────────────────┐
│           READ                       │
│  • Task file                         │
│  • .context/error-memory.md          │
│  • Related existing code             │
│  • skills/react-native/patterns.md   │
│  • 🔒 skills/security/* (nếu task    │
│      auth/storage/input/API)         │
└──────────────┬──────────────────────┘
               ▼
┌─────────────────────────────────────┐
│           PLAN                       │
│  • List files to create/modify       │
│  • Identify potential issues         │
│  • Check error-memory for pitfalls   │
└──────────────┬──────────────────────┘
               ▼
┌─────────────────────────────────────┐
│           ACT                        │
│  • Write code                        │
│  • Write tests                       │
│  • Install dependencies if needed    │
└──────────────┬──────────────────────┘
               ▼
┌─────────────────────────────────────┐
│          OBSERVE                     │
│  • Run typecheck (`npx tsc --noEmit`)│
│  • Run tests (`npx jest`)            │
│  • Run lint (`npx expo lint`)        │
│  • Check app builds                  │
└──────────────┬──────────────────────┘
               ▼
        ┌──────┴──────┐
        ▼             ▼
      PASS          FAIL
        │             │
        ▼             ▼
   git commit    Error Analyzer
        │             │
        └─────────────┘
             (retry, max 3)
```

## Task Execution Rules

1. **Read task file hoàn toàn** trước khi code
2. **Check dependencies** — chỉ chạy khi tất cả deps đã PASS
3. **Dependency check:**
   ```bash
   python3 -c "
   import json
   with open('.context/progress.json') as f:
       p = json.load(f)
   print('Completed:', p['completedTasks'])
   print('In progress:', p['inProgressTask'])
   "
   ```

4. **Update progress** sau mỗi task:
   ```bash
   python3 -c "
   import json
   with open('.context/progress.json') as f:
       p = json.load(f)
   p['completedTasks'].append('layer-N/task-NN')
   p['inProgressTask'] = None
   with open('.context/progress.json','w') as f:
       json.dump(p, f, indent=2)
   "
   ```

5. **Max 3 retries per task** — nếu vẫn FAIL → mark BLOCKED → notify human

## Verification Commands (React Native)

```bash
# TypeScript check
npx tsc --noEmit

# Lint
npx expo lint

# Unit tests
npx jest

# Run app (manual testing)
npx expo start        # Expo Go
npx expo run:ios      # iOS simulator
npx expo run:android  # Android emulator
```

## Success Criteria
- TypeScript compile pass
- Tests pass
- Lint pass
- App runs (`expo start` không lỗi)
