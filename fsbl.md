The yaml file is written in `xsctyaml.bbclass` but some of its attributes are set elsewhere e.g. `meta-xilinx-tools/recipes-bsp/device-tree/device-tree.bbappend`. Clearly the `device-tree` recipe must be run before `fsbl` receipe, but its not clear where that dependency is defined (maybe it has to be explicitly set as a dependency in local.conf?)

```mermaid
flowchart LR

%%%%%%%%%%%%%%
%% meta-xilinx
%%%%%%%%%%%%%%
subgraph meta-xilinx

%%%%%%%%%%%%%%%%%%%
%% meta-xilinx-core
%%%%%%%%%%%%%%%%%%%
    subgraph meta-xilinx-core
        subgraph meta-xilinx-core_recipes-bsp[recipes-bsp]
            subgraph meta-xilinx-core_embeddedsw[embeddedsw]
                subgraph meta-xilinx-core_fsbl.bb[fsbl.bb]
                    meta-xilinx-core_check_fsbl_variables[["check_fsbl_variables()"]]
                    meta-xilinx-core_virtual/fsbl[\"PROVIDES = virtual/fsbl"\]
                end
            end
        end    
    end

    %%%%%%%%%%%%%%%%%%%%%%%%%
    %% meta-xilinx-standalone
    %%%%%%%%%%%%%%%%%%%%%%%%%
    subgraph meta-xilinx-standalone
        subgraph meta-xilinx-standalone_recipes-bsp[recipes-bsp]
            subgraph meta-xilinx-standalone_embeddedsw[embeddedsw]
                subgraph meta-xilinx-standalone_fsbl.bbappend[fsbl.bbappend]
                    meta-xilinx-standalone_check_fsbl_variables[["check_fsbl_variables()"]]
                end
                subgraph meta-xilinx-standalone_fsbl-firmware.bb[fsbl-firmware.bb]
                    
                end
                subgraph meta-xilinx-standalone_fsbl-fw-cfg.inc[fsbl-fw-cfg.inc]
                    
                end
                subgraph meta-xilinx-standalone_fsbl-firmware.inc[fsbl-firmware.inc]
                    meta-xilinx-standalone_fsbl-firmware.inc_do_compile[["do_compile()"]]
                end                
            end
        end 
        subgraph meta-xilinx-standalone_classes[classes]
            subgraph meta-xilinx-standalone_xlnx-embeddedsw.bbclass[xlnx-embeddedsw.bbclass]
                meta-xilinx-standalone_xlnx-embeddedsw_ESW_BRANCH[\ESW_BRANCH\]
                meta-xilinx-standalone_xlnx-embeddedsw_ESW_REV[\ESW_REV\]
            end
        end   
    end
end

%%%%%%%%%%%%%%%%%%%%
%% meta-xilinx-tools
%%%%%%%%%%%%%%%%%%%%
subgraph meta-xilinx-tools
    subgraph meta-xilinx-tools_recipes-bsp[recipes-bsp]
        subgraph meta-xilinx-tools_embeddedsw[embeddedsw]
            subgraph meta-xilinx-tools_fsbl.bbappend[fsbl.bbappend]
                set-XSCTH_SCRIPT[/"XSCTH_SCRIPT = app.tcl"/]    
            end
            subgraph meta-xilinx-tools_fsbl-firmware.bbappend[fsbl-firmware.bbappend]
                do_compile:prepend:elf:aarch64[["do_compile:prepend:elf:aarch64()"]]
                do_compile:prepend:elf:arm[["do_compile:prepend:elf:arm()"]]
            end
        end        
    end    
    subgraph meta-xilinx-tools_classes[classes]
        subgraph meta-xilinx-tools_classes_xsctyaml.bbclass[xsctyaml.bbclass]
            xsctyaml-do_create_yaml[["do_create_yaml()"]]
        end
        subgraph meta-xilinx-tools_classes_xsctapp.bbclass[xsctapp.bbclass]
            subgraph meta-xilinx-tools_classes_xsctapp_do_compile["do_compile()"]
                xsctapp-oe_runmake-XSCTH_PROJ[oe_runmake XSCTH_PROJ]
            end
            subgraph meta-xilinx-tools_classes_xsctapp_do_install["do_install()"]
                xsctapp-install[install XSCTH_PROJ/*.elf /boot/*.elf]
            end
        end

        subgraph meta-xilinx-tools_classes_xsctbase.bbclass[xsctbase.bbclass]
            meta-xilinx-tools_classes_xsctbase_do_configure[["do_configure()"]]
            
        end
    end
end

local.conf -..->|depends|meta-xilinx-core_fsbl.bb

meta-xilinx-tools_fsbl.bbappend -..->|depends|meta-xilinx-standalone_fsbl-firmware.bb
meta-xilinx-standalone_fsbl.bbappend -..->|requires|meta-xilinx-standalone_fsbl-fw-cfg.inc
meta-xilinx-standalone_fsbl-firmware.bb -..->|requires|meta-xilinx-standalone_fsbl-firmware.inc
meta-xilinx-standalone_fsbl-firmware.inc -..->|inherit|meta-xilinx-standalone_xlnx-embeddedsw.bbclass
meta-xilinx-tools_classes_xsctapp.bbclass -..->|inherit|meta-xilinx-tools_classes_xsctbase.bbclass

meta-xilinx-tools_fsbl-firmware.bbappend -..->|inherit|meta-xilinx-tools_classes_xsctapp.bbclass
meta-xilinx-tools_fsbl-firmware.bbappend -..->|inherit|meta-xilinx-tools_classes_xsctyaml.bbclass
xsctyaml-do_create_yaml -->|create|fsbl-firmware.yaml
XSCTH_SCRIPT -->|reads|meta-xilinx-tools_classes_xsctbase_do_configure
vivado.xsa -->|reads|meta-xilinx-tools_classes_xsctbase_do_configure
fsbl-firmware.yaml -->|reads|meta-xilinx-tools_classes_xsctbase_do_configure

meta-xilinx-tools_classes_xsctbase_do_configure -->|output|fsbl-firmware.elf

%% fsbl recipe style
style meta-xilinx-core_fsbl.bb fill:#000099,color:white
style meta-xilinx-tools_fsbl.bbappend fill:#000099,color:white
style meta-xilinx-standalone_fsbl.bbappend fill:#000099,color:white

%% fsbl-firmware style
style meta-xilinx-standalone_fsbl-firmware.bb fill:#004d1a,color:white
style meta-xilinx-standalone_fsbl-firmware.inc fill:#004d1a,color:white
style meta-xilinx-tools_fsbl-firmware.bbappend fill:#004d1a,color:white

%% all classes
style meta-xilinx-standalone_xlnx-embeddedsw.bbclass fill:#660000,color:white
style meta-xilinx-tools_classes_xsctapp.bbclass fill:#660000,color:white
style meta-xilinx-tools_classes_xsctyaml.bbclass fill:#660000,color:white
style meta-xilinx-tools_classes_xsctbase.bbclass fill:#660000,color:white
