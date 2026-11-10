# Windows Memory  User Activity

## Tracking Sessions

#### Logged Sessions

```bash
vol -f THM-WIN-001_071528_07052025.mem windows.sessions > sessions.txt
```

#### Find Loaded Registry Hives

```
vol -f THM-WIN-001_071528_07052025.mem windows.registry.hivelist > hivelist.txt
```

```bash
vol -f THM-WIN-001_071528_07052025.mem windows.registry.userassist  > userassist  .txt
```

## Command Execution & File Access

#### Execution

```
vol -f THM-WIN-001_071528_07052025.mem windows.cmdline  > cmdline.txt
```

#### File Access

```
vol -f THM-WIN-001_071528_07052025.mem windows.handles > handles.txt
```

## Tracing User Execution

#### Investigate macro file

1. extract **WINWORD.EXE process**

```
vol -f THM-WIN-001_071528_07052025.mem -o 5252/ windows.dumpfiles --pid 5252
```

```
ls 5252/|grep dotm
    file.0x990b2ae077d0.0x990b2a3f5d70.SharedCacheMap.Normal.dotm.vacb
    file.0x990b2ae077d0.0x990b2b916cd0.DataSectionObject.Normal.dotm.dat
```

2. Confirm Macro Execution

```
olevba  file.0x990b2ae077d0.0x990b2b916cd0.DataSectionObject.Normal.dotm.dat
```

