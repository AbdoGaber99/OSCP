# Linux Memory Analysis

## Hunting for Suspicious Process

#### Identify the Correct Linux Banner

```
vol3 -f FS-01.mem linux.banners.Banners
```

#### List All Running Processes

```
vol3 -f FS-01.mem linux.pslist.PsList
```

```
vol3 -f FS-01.mem linux.psscan.PsScan
```

#### Processes With Arguments

```
vol3 -f FS-01.mem linux.psaux.PsAux
```

#### Process Mapping

```
vol3 -f FS-01.mem linux.proc.Maps
```

## Hunting for Suspicious Network Activities

#### Identify IP Information

```
vol3 -f FS-01.mem linux.ip.Addr
```

#### Identify Network Interface Information

```
vol3 -f FS-01.mem linux.ip.Link
```

#### Explore Socket Details

```
vol3 -f FS-01.mem linux.sockstat.Sockstat
```

## Hunting for User Activities

#### Bash History

```
vol3 -f FS-01.mem linux.bash.Bash
```

#### Check Environment Variables

```
vol3 -f FS-01.mem linux.envars.Envars
```
