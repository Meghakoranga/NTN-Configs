# NTN Configuration Guide: GEO and LEO Configurations

This guide outlines setting up GEO and LEO configurations for the NTN (Non-Terrestrial Network) project using OpenAirInterface (OAI). It covers gNB and UE details, configuration files, commands, and log management.

Clone the repository:
```bash
git clone https://gitlab.eurecom.fr/oai/openairinterface5g.git ~/openairinterface5g
cd ~/openairinterface5g

```

## Configuration Files
 
**gNB Configuration:**
- Path: `targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band66.fr1.25PRB.usrpx300.conf`

**LEO Channel Model:**
- Path: `targets/PROJECTS/GENERIC-NR-5GC/CONF/channelmod_rfsimu_LEO_satellite.conf`

## gNB Configuration Details
The following changes are to be made in  `targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band66.fr1.25PRB.usrpx300.conf` 
### Key Parameters

**cellSpecificKoffset**: Adjusts for NTN propagation delay. Set `cellSpecificKoffset_r17` in `servingCellConfigCommon`:
```bash
      cellSpecificKoffset_r17 = 478; # GEO
#      cellSpecificKoffset_r17 = 40; # LEO
```

**Timers**: Extend timers in `gNBs.[0].TIMERS` for GEO:
```bash
    TIMERS :
    {
      sr_ProhibitTimer       = 0;
      sr_TransMax            = 64;
      sr_ProhibitTimer_v1700 = 512;
      t300                   = 2000;
      t301                   = 2000;
      t319                   = 2000;
    };
```

**HARQ Feedback**: For better throughput with large RTT, add `disable_harq`:
```bash
    disable_harq = 1; // <---
```

## Simulation Parameters

### GEO Simulation
Add the following to gNB and UE command lines:
```bash
--rfsimulator.prop_delay 238.74
```

### LEO Simulation
Two models:
- **SAT_LEO_TRANS**: transparent satellite, gNB on ground
- **SAT_LEO_REGEN**: regenerative satellite, gNB on board

Example for transparent LEO:
```bash
channelmod = {
  max_chan=10;
  modellist="modellist_rfsimu_1";
  modellist_rfsimu_1 = (
    {
      model_name     = "rfsimu_channel_enB0"
      type           = "SAT_LEO_TRANS";
      noise_power_dB = -100;
    },
    {
      model_name     = "rfsimu_channel_ue0"
      type           = "SAT_LEO_TRANS";
      noise_power_dB = -100;
    }
  );
};
```
Add to conf file `targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band66.fr1.25PRB.usrpx300.conf` under `rfsimulator`:
```bash
options = ("chanmod");
```
Or use command line:
```bash
--rfsimulator.options chanmod
```

To simulate a LEO satellite channel model with rfsimulator in UL (DL is simulated at the UE side), either the channelmod section as shown before has to be added to the gNB conf file, or a channelmod conf file has to be included like this:
```bash
@include "channelmod_rfsimu_LEO_satellite.conf"
```

## Commands
After configuring the necessary configuration files, you can run the following commands:


 **GEO GNB :**
```bash
cd cmake_targets
sudo ./ran_build/build/nr-softmodem -O ../targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band66.fr1.25PRB.usrpx300.conf --rfsim --rfsimulator.prop_delay 238.74
```
**GEO UE  :**
```bash
sudo ./ran_build/build/nr-uesoftmodem --band 66 -C 2152680000 --CO -400000000 -r 25 --numerology 0 --ssb 48 --rfsim --rfsimulator.prop_delay 238.74 --ntn-koffset 478 --ntn-ta-common 477.48
```


 **LEO GNB  :**
```bash
cd cmake_targets
sudo ./ran_build/build/nr-softmodem -O ../targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band66.fr1.25PRB.usrpx300.conf --rfsim --rfsimulator.prop_delay 20
```


 **LEO UE  :**
```bash
sudo ./ran_build/build/nr-uesoftmodem --band 66 -C 2152680000 --CO -400000000 -r 25 --numerology 0 --ssb 48 --rfsim --rfsimulator.prop_delay 20 --rfsimulator.options chanmod -O ../targets/PROJECTS/GENERIC-NR-5GC/CONF/channelmod_rfsimu_LEO_satellite.conf --time-sync-I 0.2 --ntn-koffset 40 --ntn-ta-common 37.74 --ntn-ta-commondrift -50 --autonomous-ta
```
## Commands for Saving Logs in GEO and LEO Configurations
 
If you wish to save logs while running the commands for better analysis and troubleshooting, you can use the  following commands .


#### 1. **GEO GNB  :**
```bash
cd cmake_targets
sudo ./ran_build/build/nr-softmodem -O ../targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band66.fr1.25PRB.usrpx300.conf --rfsim --rfsimulator.prop_delay 238.74 > gnb_geo.log 2>&1
```

#### 2. **LEO GNB  :**
```bash
cd cmake_targets
sudo ./ran_build/build/nr-softmodem -O ../targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band66.fr1.25PRB.usrpx300.conf --rfsim --rfsimulator.prop_delay 20 > gnb_leo.log 2>&1
```

#### 3. **GEO UE  :**
```bash
sudo ./ran_build/build/nr-uesoftmodem --band 66 -C 2152680000 --CO -400000000 -r 25 --numerology 0 --ssb 48 --rfsim --rfsimulator.prop_delay 238.74 --ntn-koffset 478 --ntn-ta-common 477.48 > ue_geo.log 2>&1
```

#### 4. **LEO UE  :**
```bash
sudo ./ran_build/build/nr-uesoftmodem --band 66 -C 2152680000 --CO -400000000 -r 25 --numerology 0 --ssb 48 --rfsim --rfsimulator.prop_delay 20 --rfsimulator.options chanmod -O ../targets/PROJECTS/GENERIC-NR-5GC/CONF/channelmod_rfsimu_LEO_satellite.conf --time-sync-I 0.2 --ntn-koffset 40 --ntn-ta-common 37.74 --ntn-ta-commondrift -50 --autonomous-ta > ue_leo.log 2>&1
```
 The commands above include options to redirect both standard output and errors to log files.



