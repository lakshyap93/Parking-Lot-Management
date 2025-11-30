    simport React, { useState, useEffect, useRef } from 'react';
import { 
  Monitor, 
  Cpu, 
  HardDrive, 
  Play, 
  Square, 
  Settings, 
  Plus, 
  Terminal, 
  Activity, 
  X, 
  Maximize2,
  Minimize2,
  Power
} from 'lucide-react';

// --- Mock Data & Constants ---
const MOCK_LOGS_WINDOWS = [
  "BIOS Date 01/01/25 15:22:11 Ver: 08.00.15",
  "CPU: TechnoVm Virtual Core i9 @ 5.2GHz",
  "Memory Test: 16384K OK",
  "Detecting Primary Master ... Techno Virtual Disk",
  "Detecting Primary Slave ... None",
  "Booting from Hard Disk...",
  "Loading Kernel...",
  "Verifying DMI Pool Data...",
  "Starting Windows Session Manager...",
  "Initializing Video Driver...",
  "Welcome to Techno OS Environment"
];

const MOCK_LOGS_LINUX = [
  "[  0.000000] Linux version 6.5.0-techno-generic",
  "[  0.234120] Command line: BOOT_IMAGE=/boot/vmlinuz",
  "[  0.551230] x86/fpu: Supporting XSAVE feature 0x001",
  "[  1.002340] TechnoVm: Virtualization extensions detected",
  "[  1.200000] Run /init as init process",
  "[  2.100000] systemd[1]: Detected architecture x86-64.",
  "[  2.500000] [ OK ] Reached target Basic System.",
  "[  3.100000] [ OK ] Started Network Service.",
  "[  3.800000] [ OK ] Started Display Manager.",
  "TechnoVm Linux is ready."
];

const OS_PRESETS = [
  { id: 1, name: 'Windows 11 Pro', type: 'windows', icon: '🪟', storage: '64 GB', ram: '8 GB', status: 'Stopped' },
  { id: 2, name: 'Ubuntu 24.04', type: 'linux', icon: '🐧', storage: '32 GB', ram: '4 GB', status: 'Stopped' },
  { id: 3, name: 'MacOS Sonoma', type: 'mac', icon: '🍎', storage: '128 GB', ram: '16 GB', status: 'Stopped' },
  { id: 4, name: 'Android x86', type: 'android', icon: '🤖', storage: '16 GB', ram: '2 GB', status: 'Stopped' },
];

// --- Main Component ---
export default function TechnoVm() {
  const [vms, setVms] = useState(OS_PRESETS);
  const [activeVM, setActiveVM] = useState(null); // The VM currently being interacted with
  const [bootState, setBootState] = useState('off'); // off, booting, running
  const [bootLogs, setBootLogs] = useState([]);
  const [showModal, setShowModal] = useState(false);
  
  // Resource Simulation
  const [cpuUsage, setCpuUsage] = useState(0);
  const [ramUsage, setRamUsage] = useState(0);

  // --- Effects ---

  // Resource Monitor Animation
  useEffect(() => {
    const interval = setInterval(() => {
      if (bootState === 'running' || bootState === 'booting') {
        setCpuUsage(Math.floor(Math.random() * 40) + 10);
        setRamUsage(Math.floor(Math.random() * 30) + 20);
      } else {
        setCpuUsage(0);
        setRamUsage(0);
      }
    }, 1000);
    return () => clearInterval(interval);
  }, [bootState]);

  // Boot Sequence Logic
  useEffect(() => {
    if (bootState === 'booting' && activeVM) {
      let logs = activeVM.type === 'windows' ? MOCK_LOGS_WINDOWS : MOCK_LOGS_LINUX;
      let currentIndex = 0;
      setBootLogs([]);

      const bootInterval = setInterval(() => {
        if (currentIndex < logs.length) {
          setBootLogs(prev => [...prev, logs[currentIndex]]);
          currentIndex++;
        } else {
          clearInterval(bootInterval);
          setTimeout(() => setBootState('running'), 1000);
        }
      }, 600); // Speed of text scrolling

      return () => clearInterval(bootInterval);
    }
  }, [bootState, activeVM]);

  // --- Handlers ---

  const handleStartVM = (vm) => {
    setActiveVM(vm);
    setBootState('booting');
    
    // Update list status
    setVms(prev => prev.map(v => v.id === vm.id ? {...v, status: 'Running'} : v));
  };

  const handleStopVM = () => {
    setBootState('off');
    setBootLogs([]);
    setVms(prev => prev.map(v => v.id === activeVM?.id ? {...v, status: 'Stopped'} : v));
    setActiveVM(null);
  };

  const handleCreateVM = (e) => {
    e.preventDefault();
    const name = e.target.vmName.value;
    const type = e.target.osType.value;
    const newId = vms.length + 1;
    
    let icon = '🖥️';
    if(type.includes('Win')) icon = '🪟';
    if(type.includes('Lin') || type.includes('Ubu')) icon = '🐧';
    if(type.includes('Mac')) icon = '🍎';

    const newVM = {
      id: newId,
      name: name || `New VM ${newId}`,
      type: 'custom',
      icon: icon,
      storage: '32 GB',
      ram: '4 GB',
      status: 'Stopped'
    };
    
    setVms([...vms, newVM]);
    setShowModal(false);
  };

  return (
    <div className="flex h-screen w-full bg-slate-900 text-slate-100 font-sans overflow-hidden selection:bg-cyan-500 selection:text-white">
      
      {/* Sidebar */}
      <aside className="w-20 md:w-64 bg-slate-950 border-r border-slate-800 flex flex-col justify-between transition-all duration-300">
        <div>
          <div className="h-16 flex items-center justify-center md:justify-start md:px-6 border-b border-slate-800">
            <div className="w-8 h-8 bg-gradient-to-tr from-cyan-500 to-blue-600 rounded-lg flex items-center justify-center shadow-lg shadow-cyan-500/20">
              <Cpu size={20} className="text-white" />
            </div>
            <span className="ml-3 font-bold text-xl tracking-wide hidden md:block bg-gradient-to-r from-white to-slate-400 bg-clip-text text-transparent">
              TechnoVm
            </span>
          </div>

          <nav className="mt-8 px-2 md:px-4 space-y-2">
            <NavItem icon={<Monitor size={20} />} label="Dashboard" active />
            <NavItem icon={<HardDrive size={20} />} label="Storage" />
            <NavItem icon={<Activity size={20} />} label="Performance" />
            <NavItem icon={<Settings size={20} />} label="Settings" />
          </nav>
        </div>

        <div className="p-4 border-t border-slate-800 hidden md:block">
          <div className="bg-slate-900 rounded-lg p-3 border border-slate-800">
            <div className="text-xs text-slate-400 mb-2 font-medium">HOST RESOURCES</div>
            <div className="space-y-3">
              <ResourceBar label="CPU" percent={cpuUsage + 5} color="bg-cyan-500" />
              <ResourceBar label="RAM" percent={ramUsage + 10} color="bg-purple-500" />
            </div>
          </div>
        </div>
      </aside>

      {/* Main Content Area */}
      <main className="flex-1 flex flex-col relative overflow-hidden">
        
        {/* Top Header */}
        <header className="h-16 bg-slate-900/50 backdrop-blur-md border-b border-slate-800 flex items-center justify-between px-6 z-10">
          <h2 className="text-lg font-medium text-slate-200">Virtual Machines</h2>
          <button 
            onClick={() => setShowModal(true)}
            className="bg-cyan-600 hover:bg-cyan-500 text-white px-4 py-2 rounded-md flex items-center gap-2 text-sm font-medium transition-colors shadow-lg shadow-cyan-900/20"
          >
            <Plus size={16} />
            <span className="hidden sm:inline">New Machine</span>
          </button>
        </header>

        {/* Workspace */}
        <div className="flex-1 overflow-y-auto p-6 relative">
          
          {/* Active VM View (Overlay) */}
          {activeVM && (
            <div className="absolute inset-0 z-20 bg-slate-900 p-4 flex flex-col animate-in fade-in duration-300">
              {/* VM Window Chrome */}
              <div className="flex-1 bg-black rounded-lg border border-slate-700 shadow-2xl flex flex-col overflow-hidden ring-1 ring-slate-800">
                {/* VM Title Bar */}
                <div className="h-10 bg-slate-800 flex items-center justify-between px-4 select-none">
                  <div className="flex items-center gap-2">
                    <span className="text-lg">{activeVM.icon}</span>
                    <span className="text-sm font-medium text-slate-200">{activeVM.name} - TechnoVm</span>
                  </div>
                  <div className="flex items-center gap-3">
                    <div className="flex gap-2 mr-4">
                      <div className="w-2 h-2 rounded-full bg-green-500 animate-pulse"></div>
                      <span className="text-xs text-slate-400">Running</span>
                    </div>
                    <button className="text-slate-400 hover:text-white"><Minimize2 size={16} /></button>
                    <button className="text-slate-400 hover:text-white"><Maximize2 size={16} /></button>
                    <button onClick={handleStopVM} className="text-red-400 hover:text-red-300"><X size={16} /></button>
                  </div>
                </div>

                {/* VM Viewport */}
                <div className="flex-1 relative bg-black font-mono text-slate-300">
                  
                  {/* Boot Screen */}
                  {bootState === 'booting' && (
                    <div className="p-8 h-full flex flex-col">
                      <div className="w-full max-w-2xl">
                        {bootLogs.map((log, i) => (
                          <div key={i} className="mb-1 text-sm md:text-base text-green-500">
                            <span className="text-slate-500 mr-2">{`>`}</span>
                            {log}
                          </div>
                        ))}
                        <div className="animate-pulse text-green-500 mt-2">_</div>
                      </div>
                    </div>
                  )}

                  {/* Running OS Screen (Simulated) */}
                  {bootState === 'running' && (
                    <div className="h-full w-full relative bg-cover bg-center flex flex-col" 
                         style={{ 
                           background: activeVM.type === 'windows' 
                             ? 'linear-gradient(135deg, #0078d7 0%, #0062c4 100%)' 
                             : activeVM.type === 'linux' ? '#2c001e' : '#e0e0e0'
                         }}>
                      
                      {/* Fake OS Desktop */}
                      <div className="flex-1 flex items-center justify-center">
                         <div className="text-center">
                            <div className="text-6xl mb-4 drop-shadow-lg animate-bounce">{activeVM.icon}</div>
                            <h1 className={`text-3xl font-bold drop-shadow-md ${activeVM.type === 'mac' ? 'text-slate-800' : 'text-white'}`}>
                              {activeVM.name}
                            </h1>
                            <p className={`mt-2 opacity-80 ${activeVM.type === 'mac' ? 'text-slate-700' : 'text-slate-200'}`}>
                              System initialized successfully.
                            </p>
                         </div>
                      </div>

                      {/* Fake Taskbar */}
                      <div className={`h-12 w-full flex items-center px-4 gap-4 ${activeVM.type === 'mac' ? 'bg-white/50 backdrop-blur-xl mb-4 mx-auto w-auto rounded-xl shadow-lg self-center' : 'bg-black/40 backdrop-blur-sm'}`}>
                         <div className="w-8 h-8 bg-white/20 rounded hover:bg-white/40 transition-colors cursor-pointer"></div>
                         <div className="w-8 h-8 bg-blue-500/20 rounded hover:bg-blue-500/40 transition-colors cursor-pointer"></div>
                         <div className="w-8 h-8 bg-green-500/20 rounded hover:bg-green-500/40 transition-colors cursor-pointer"></div>
                      </div>
                    </div>
                  )}
                </div>
              </div>
            </div>
          )}

          {/* VM Grid (Dashboard) */}
          <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
            {vms.map((vm) => (
              <div key={vm.id} className="bg-slate-800/50 border border-slate-700 rounded-xl p-5 hover:border-cyan-500/50 hover:shadow-lg hover:shadow-cyan-500/10 transition-all group">
                <div className="flex justify-between items-start mb-4">
                  <div className="w-12 h-12 bg-slate-700 rounded-lg flex items-center justify-center text-2xl group-hover:scale-110 transition-transform">
                    {vm.icon}
                  </div>
                  <div className={`px-2 py-1 rounded text-xs font-semibold ${vm.status === 'Running' ? 'bg-green-500/20 text-green-400' : 'bg-slate-700 text-slate-400'}`}>
                    {vm.status}
                  </div>
                </div>
                
                <h3 className="text-lg font-bold text-slate-100">{vm.name}</h3>
                <div className="mt-4 space-y-2">
                  <div className="flex justify-between text-sm">
                    <span className="text-slate-400">Storage</span>
                    <span className="text-slate-200">{vm.storage}</span>
                  </div>
                  <div className="flex justify-between text-sm">
                    <span className="text-slate-400">Memory</span>
                    <span className="text-slate-200">{vm.ram}</span>
                  </div>
                </div>

                <div className="mt-6 flex gap-2">
                  <button 
                    onClick={() => handleStartVM(vm)}
                    disabled={vm.status === 'Running'}
                    className="flex-1 bg-cyan-600 hover:bg-cyan-500 disabled:opacity-50 disabled:cursor-not-allowed text-white py-2 rounded-lg font-medium text-sm transition-colors flex items-center justify-center gap-2"
                  >
                    <Play size={16} fill="currentColor" />
                    Start
                  </button>
                  <button className="px-3 bg-slate-700 hover:bg-slate-600 rounded-lg text-slate-300 transition-colors">
                    <Settings size={18} />
                  </button>
                </div>
              </div>
            ))}
          </div>

        </div>
      </main>

      {/* New VM Modal */}
      {showModal && (
        <div className="fixed inset-0 z-50 flex items-center justify-center bg-black/60 backdrop-blur-sm p-4">
          <div className="bg-slate-900 border border-slate-700 rounded-xl shadow-2xl w-full max-w-md overflow-hidden animate-in zoom-in-95 duration-200">
            <div className="p-4 border-b border-slate-800 flex justify-between items-center bg-slate-800/50">
              <h3 className="font-bold text-white">Create Virtual Machine</h3>
              <button onClick={() => setShowModal(false)} className="text-slate-400 hover:text-white"><X size={20} /></button>
            </div>
            <form onSubmit={handleCreateVM} className="p-6 space-y-4">
              <div>
                <label className="block text-sm font-medium text-slate-400 mb-1">Name</label>
                <input 
                  name="vmName" 
                  type="text" 
                  placeholder="e.g. My Server" 
                  className="w-full bg-slate-950 border border-slate-700 rounded-lg px-3 py-2 text-white focus:outline-none focus:border-cyan-500"
                  required 
                />
              </div>
              <div>
                <label className="block text-sm font-medium text-slate-400 mb-1">Operating System</label>
                <select name="osType" className="w-full bg-slate-950 border border-slate-700 rounded-lg px-3 py-2 text-white focus:outline-none focus:border-cyan-500">
                  <option value="Windows 11">Windows</option>
                  <option value="Ubuntu">Linux (Ubuntu)</option>
                  <option value="MacOS">macOS</option>
                  <option value="Android">Android</option>
                </select>
              </div>
              <div className="grid grid-cols-2 gap-4">
                 <div>
                  <label className="block text-sm font-medium text-slate-400 mb-1">RAM</label>
                  <select className="w-full bg-slate-950 border border-slate-700 rounded-lg px-3 py-2 text-white focus:outline-none focus:border-cyan-500">
                    <option>4 GB</option>
                    <option>8 GB</option>
                    <option>16 GB</option>
                  </select>
                 </div>
                 <div>
                  <label className="block text-sm font-medium text-slate-400 mb-1">Disk</label>
                  <select className="w-full bg-slate-950 border border-slate-700 rounded-lg px-3 py-2 text-white focus:outline-none focus:border-cyan-500">
                    <option>64 GB</option>
                    <option>128 GB</option>
                    <option>512 GB</option>
                  </select>
                 </div>
              </div>
              <div className="pt-4 flex justify-end gap-3">
                <button type="button" onClick={() => setShowModal(false)} className="px-4 py-2 text-slate-300 hover:text-white transition-colors">Cancel</button>
                <button type="submit" className="px-4 py-2 bg-cyan-600 hover:bg-cyan-500 text-white rounded-lg font-medium transition-colors">Create</button>
              </div>
            </form>
          </div>
        </div>
      )}

    </div>
  );
}

// --- Helper Components ---

function NavItem({ icon, label, active }) {
  return (
    <button className={`w-full flex items-center gap-3 px-3 py-3 rounded-lg transition-colors ${active ? 'bg-cyan-500/10 text-cyan-400 border border-cyan-500/20' : 'text-slate-400 hover:bg-slate-800 hover:text-slate-200'}`}>
      {icon}
      <span className="font-medium hidden md:block">{label}</span>
    </button>
  );
}

function ResourceBar({ label, percent, color }) {
  return (
    <div>
      <div className="flex justify-between text-xs text-slate-400 mb-1">
        <span>{label}</span>
        <span>{percent}%</span>
      </div>
      <div className="h-1.5 w-full bg-slate-800 rounded-full overflow-hidden">
        <div 
          className={`h-full ${color} transition-all duration-500`} 
          style={{ width: `${percent}%` }}
        />
      </div>
    </div>
  );
}

tatic void showAvailableSlots() {
        int carUsed = countType("Car");
        int bikeUsed = countType("Bike");
        System.out.println(" Car slots available: " + (totalCarSlots - carUsed));
        System.out.println(" Bike slots available: " + (totalBikeSlots - bikeUsed));
    }

    static void showParkedVehicles() {
        if (parkedVehicles.isEmpty()) {
            System.out.println("No vehicles currently parked.");
            return;
        }
        System.out.println("Currently Parked Vehicles:");
        for (Vehicle v : parkedVehicles.values()) {
            System.out.println(" - " + v.type + " | Number: " + v.number);
        }
    }

    static int countType(String type) {
        int count = 0;
        for (Vehicle v : parkedVehicles.values()) {
            if (v.type.equalsIgnoreCase(type)) count++;
        }
        return count;
    }
}
