# HTB Write-ups

My Hack The Box write-ups, as I go. Notes, tools, and struggles included.

Each write-up covers the concepts I ran into, the tools I used, and where I got stuck, so I can track my progress over time.

> I only publish write-ups for **retired** machines, and never include flags or hashes, in line with HTB rules.

## Progress

| # | Machine | OS | Difficulty | Key concepts | Date | Write-up |
|---|---------|----|-----------|--------------|------|----------|
| 1 | Cap    | Linux   | Easy | IDOR, pcap analysis, FTP/SSH, Linux capabilities | 2026-11-11 | [link](boxes/cap.md) |
| 2 | Blue   | Windows | Easy | SMB, EternalBlue (MS17-010) | 2026-11-11 | [link](boxes/blue.md) |
| 3 | Legacy | Windows | Easy | SMB, MS08-067, MS17-010 | 2026-11-12 | [link](boxes/legacy.md) |
| 4 | Lame   | Linux   | Easy | Samba usermap_script, distcc, Metasploit | 2026-11-12 | [link](boxes/lame.md) |

## Notes

- [Tools I've used](notes/tools.md)
- [Concepts I've learned](notes/concepts.md)

## Structure

```
boxes/        one write-up per machine
notes/        tools, concepts, cheatsheets
assets/       screenshots
TEMPLATE.md   template to copy for each new machine
```

## Goals

- [ ] Finish the Easy retired machines
- [ ] Move on to Medium machines
- [ ] Go from automated tools (Metasploit) to manual exploitation
