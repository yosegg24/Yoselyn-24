<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Control de Servicios y Comisiones en Bs</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/alpinejs/3.13.3/cdn.min.js" defer></script>
</head>
<body class="bg-slate-50 text-slate-800 font-sans" x-data="salonApp()">
    
    <div class="max-w-6xl mx-auto p-4 md:p-6">
        <!-- Header -->
        <header class="bg-white rounded-2xl shadow-sm p-6 mb-6 flex flex-col md:flex-row justify-between items-center gap-4">
            <div>
                <h1 class="text-2xl font-bold text-emerald-700">Control de Servicios y Comisiones en Bs</h1>
                <p class="text-sm text-slate-500">Gestión en Bolívares, métodos nacionales y cierres semanales</p>
            </div>
            <div class="flex flex-wrap gap-2 w-full md:w-auto items-center">
                <!-- Global Euro Rate Setting -->
                <div class="bg-blue-50 border border-blue-200 px-3 py-1.5 rounded-xl flex items-center gap-2">
                    <span class="text-xs font-bold text-blue-900">💶 Tasa Euro BCV:</span>
                    <input type="number" step="0.01" x-model="dailyEuroRate" @input="saveGlobalRate()" placeholder="1001.18" class="w-24 p-1 border rounded text-xs bg-white text-center font-bold text-blue-900 focus:ring-2 focus:ring-blue-500">
                </div>
                <input type="text" x-model="newEmployeeName" @keyup.enter="addEmployee()" placeholder="Nombre de empleada" class="px-3 py-2 border rounded-xl text-sm focus:outline-none focus:ring-2 focus:ring-emerald-500">
                <button @click="addEmployee()" class="bg-emerald-600 hover:bg-emerald-700 text-white px-3 py-2 rounded-xl text-sm font-semibold transition whitespace-nowrap">+ Empleada</button>
                <button @click="openArchiveModal = true" class="bg-indigo-600 hover:bg-indigo-700 text-white px-3 py-2 rounded-xl text-sm font-semibold transition whitespace-nowrap">📁 Historial</button>
            </div>
        </header>

        <!-- Navigation Tabs & Reset Button -->
        <div class="flex overflow-x-auto gap-2 mb-6 pb-2 border-b justify-between items-center">
            <div class="flex gap-2 overflow-x-auto pb-2">
                <template x-for="(emp, index) in employees" :key="index">
                    <div class="flex items-center" :class="currentTab === index ? 'bg-emerald-600 text-white shadow-md' : 'bg-white text-slate-600 hover:bg-slate-100 border'" style="border-radius: 0.75rem;">
                        <button @click="currentTab = index" class="px-4 py-2.5 font-medium text-sm transition whitespace-nowrap flex items-center gap-2">
                            <span x-text="emp.name"></span>
                            <span class="text-xs px-2 py-0.5 rounded-full" :class="currentTab === index ? 'bg-emerald-800 text-white' : 'bg-slate-200 text-slate-700'" x-text="emp.services.length"></span>
                        </button>
                        <button @click.stop="deleteEmployee(index)" class="pr-3 pl-1 text-xs opacity-70 hover:opacity-100" title="Eliminar empleada">✕</button>
                    </div>
                </template>
            </div>
            <button @click="closeWeek()" class="bg-amber-600 hover:bg-amber-700 text-white px-4 py-2.5 rounded-xl text-sm font-semibold transition whitespace-nowrap shadow-sm">🔄 Cerrar Semana y Reiniciar</button>
        </div>

        <!-- Empty State Notice -->
        <template x-if="employees.length === 0">
            <div class="bg-white rounded-2xl p-12 text-center shadow-sm border border-slate-100 my-6">
                <h3 class="text-lg font-bold text-slate-700 mb-2">No hay empleadas registradas</h3>
                <p class="text-slate-500 text-sm mb-4">Escribe el nombre de tu primera empleada arriba y presiona "+ Empleada" para comenzar.</p>
            </div>
        </template>

        <!-- Active Employee Tab Content -->
        <template x-for="(emp, index) in employees" :key="index">
            <div x-show="currentTab === index" class="space-y-6">
                
                <!-- Summary KPI Cards -->
                <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
                    <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-100">
                        <p class="text-xs font-semibold text-slate-400 uppercase tracking-wider">Total Servicios</p>
                        <h3 class="text-2xl font-bold text-slate-800 mt-1" x-text="emp.services.length"></h3>
                    </div>
                    <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-100">
                        <p class="text-xs font-semibold text-slate-400 uppercase tracking-wider">Ventas Totales (Bs.)</p>
                        <h3 class="text-xl font-bold text-emerald-600 mt-1" x-text="formatVES(calculateTotalVES(emp))"></h3>
                        <p class="text-xs text-slate-400 mt-0.5" x-text="'≈ €' + (calculateTotalVES(emp) / (dailyEuroRate || 1)).toFixed(2)"></p>
                    </div>
                    <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-100">
                        <p class="text-xs font-semibold text-slate-400 uppercase tracking-wider">Comisiones Totales (Bs.)</p>
                        <h3 class="text-xl font-bold text-indigo-600 mt-1" x-text="formatVES(calculateTotalCommissionVES(emp))"></h3>
                        <p class="text-xs text-slate-400 mt-0.5" x-text="'≈ €' + (calculateTotalCommissionVES(emp) / (dailyEuroRate || 1)).toFixed(2)"></p>
                    </div>
                </div>

                <!-- Add Service Form & Table Grid -->
                <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                    
                    <!-- Form Card -->
                    <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-100 h-fit">
                        <h3 class="font-bold text-lg text-slate-800 mb-4">Registrar Nuevo Servicio</h3>
                        <form @submit.prevent="addService(index)" class="space-y-4">
                            <div>
                                <label class="block text-xs font-semibold text-slate-500 mb-1">Fecha</label>
                                <input type="date" x-model="form.date" required class="w-full p-2.5 border rounded-xl text-sm focus:ring-2 focus:ring-emerald-500">
                            </div>
                            <div>
                                <label class="block text-xs font-semibold text-slate-500 mb-1">Nombre de la Clienta</label>
                                <input type="text" x-model="form.client" placeholder="Ej. María Pérez" required class="w-full p-2.5 border rounded-xl text-sm focus:ring-2 focus:ring-emerald-500">
                            </div>
                            <div>
                                <label class="block text-xs font-semibold text-slate-500 mb-1">Servicio Realizado</label>
                                <input type="text" x-model="form.service" placeholder="Ej. Manicura / Balayage" required class="w-full p-2.5 border rounded-xl text-sm focus:ring-2 focus:ring-emerald-500">
                            </div>
                            
                            <div>
                                <label class="block text-xs font-semibold text-slate-500 mb-1">Monto en Bolívares (Bs.)</label>
                                <input type="number" step="0.01" x-model="form.priceVES" placeholder="0.00" required class="w-full p-2.5 border rounded-xl text-sm focus:ring-2 focus:ring-emerald-500 font-bold text-emerald-700">
                                <p class="text-xs text-slate-400 mt-1" x-show="form.priceVES">
                                    Equivalente aprox: <strong x-text="'€' + (form.priceVES / (dailyEuroRate || 1)).toFixed(2)"></strong>
                                </p>
                            </div>

                            <div>
                                <label class="block text-xs font-semibold text-slate-500 mb-1">Método de Pago</label>
                                <select x-model="form.payment" required class="w-full p-2.5 border rounded-xl text-sm focus:ring-2 focus:ring-emerald-500 bg-white font-medium">
                                    <option value="Efectivo Bs.">Efectivo (Bs.)</option>
                                    <option value="Pago Móvil">Pago Móvil</option>
                                    <option value="Punto de Venta">Punto de Venta</option>
                                    <option value="Transferencia Bancaria">Transferencia Bancaria</option>
                                </select>
                            </div>

                            <div>
                                <label class="block text-xs font-semibold text-slate-500 mb-1">% Comisión</label>
                                <input type="number" x-model="form.commissionRate" required class="w-full p-2.5 border rounded-xl text-sm focus:ring-2 focus:ring-emerald-500">
                            </div>
                            <button type="submit" class="w-full bg-emerald-600 hover:bg-emerald-700 text-white font-semibold py-3 rounded-xl transition shadow-sm">Registrar Servicio</button>
                        </form>
                    </div>

                    <!-- Table List Card -->
                    <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-100 lg:col-span-2 overflow-x-auto">
                        <div class="flex justify-between items-center mb-4">
                            <h3 class="font-bold text-lg text-slate-800" x-text="'Historial Actual de ' + emp.name"></h3>
                            <button @click="shareWhatsApp(emp)" class="bg-emerald-50 text-emerald-700 hover:bg-emerald-100 px-3 py-1.5 rounded-lg text-xs font-semibold transition flex items-center gap-1.5">
                                💬 Compartir por WhatsApp
                            </button>
                        </div>
                        
                        <table class="w-full text-left border-collapse">
                            <thead>
                                <tr class="border-b text-xs font-semibold text-slate-400 uppercase tracking-wider">
                                    <th class="pb-3">Fecha</th>
                                    <th class="pb-3">Clienta</th>
                                    <th class="pb-3">Servicio</th>
                                    <th class="pb-3">Precio (Bs.)</th>
                                    <th class="pb-3">Pago</th>
                                    <th class="pb-3">Comisión (Bs.)</th>
                                    <th class="pb-3 text-center">Acción</th>
                                </tr>
                            </thead>
                            <tbody class="divide-y text-sm">
                                <template x-for="(s, sIndex) in emp.services" :key="sIndex">
                                    <tr class="hover:bg-slate-50">
                                        <td class="py-3 text-slate-500 text-xs" x-text="s.date"></td>
                                        <td class="py-3 font-medium text-slate-800" x-text="s.client"></td>
                                        <td class="py-3 text-slate-600" x-text="s.service"></td>
                                        <td class="py-3 font-bold text-emerald-700" x-text="formatVES(s.priceVES)"></td>
                                        <td class="py-3">
                                            <span class="text-xs px-2 py-1 bg-slate-100 rounded-md font-medium text-slate-600" x-text="s.payment"></span>
                                        </td>
                                        <td class="py-3 font-bold text-indigo-600" x-text="formatVES(s.priceVES * (s.commissionRate / 100))"></td>
                                        <td class="py-3 text-center">
                                            <button @click="deleteService(index, sIndex)" class="text-rose-500 hover:text-rose-700 text-xs font-semibold">Eliminar</button>
                                        </td>
                                    </tr>
                                </template>
                                <template x-if="emp.services.length === 0">
                                    <tr>
                                        <td colspan="7" class="py-8 text-center text-slate-400 text-sm">No hay servicios registrados esta semana.</td>
                                    </tr>
                                </template>
                            </tbody>
                        </table>
                    </div>

                </div>
            </div>
        </template>

        <!-- Archive Modal -->
        <div x-show="openArchiveModal" class="fixed inset-0 bg-black/50 flex items-center justify-center p-4 z-50" style="display: none;">
            <div class="bg-white rounded-2xl max-w-2xl w-full p-6 max-h-[85vh] overflow-y-auto shadow-xl">
                <div class="flex justify-between items-center mb-4 border-b pb-3">
                    <h2 class="text-xl font-bold text-slate-800">📁 Registros Pasados (Historial de Semanas)</h2>
                    <button @click="openArchiveModal = false" class="text-slate-400 hover:text-slate-600 font-bold text-lg">✕</button>
                </div>
                
                <template x-for="(week, wIndex) in archives" :key="wIndex">
                    <div class="mb-6 bg-slate-50 p-4 rounded-xl border border-slate-200">
                        <div class="flex justify-between items-center mb-2">
                            <h4 class="font-bold text-emerald-700" x-text="'Semana: ' + week.weekTitle"></h4>
                            <span class="text-xs text-slate-500" x-text="'Cerrado el: ' + week.closedDate"></span>
                        </div>
                        <template x-for="empData in week.data">
                            <div class="mb-3 bg-white p-3 rounded-lg border">
                                <div class="font-semibold text-sm text-slate-700 mb-1" x-text="empData.name + ' (Servicios: ' + empData.services.length + ') '"></div>
                                <div class="text-xs text-slate-500">
                                    Total Ventas: <span class="font-bold text-emerald-600" x-text="formatVES(empData.services.reduce((a,b)=>a+b.priceVES,0))"></span> | 
                                    Comisiones: <span class="font-bold text-indigo-600" x-text="formatVES(empData.services.reduce((a,b)=>a+(b.priceVES*(b.commissionRate/100)),0))"></span>
                                </div>
                            </div>
                        </template>
                    </div>
                </template>
                
                <template x-if="archives.length === 0">
                    <p class="text-center text-slate-400 py-8">No hay semanas archivadas todavía.</p>
                </template>
            </div>
        </div>
    </div>

    <script>
        function salonApp() {
            return {
                currentTab: 0,
                newEmployeeName: '',
                openArchiveModal: false,
                dailyEuroRate: parseFloat(localStorage.getItem('salon_euro_rate')) || 1001.18,
                form: {
                    date: new Date().toISOString().split('T')[0],
                    client: '',
                    service: '',
                    priceVES: '',
                    payment: 'Pago Móvil',
                    commissionRate: 40
                },
                employees: JSON.parse(localStorage.getItem('salon_app_data_bs_only')) || [],
                archives: JSON.parse(localStorage.getItem('salon_app_archives_bs_only')) || [],
                
                saveGlobalRate() {
                    localStorage.setItem('salon_euro_rate', this.dailyEuroRate);
                },
                addEmployee() {
                    if(!this.newEmployeeName.trim()) return;
                    this.employees.push({ name: this.newEmployeeName.trim(), services: [] });
                    this.newEmployeeName = '';
                    this.saveData();
                    this.currentTab = this.employees.length - 1;
                },
                deleteEmployee(index) {
                    if(confirm('¿Estás segura de eliminar esta empleada y todos sus servicios?')) {
                        this.employees.splice(index, 1);
                        this.saveData();
                        if(this.currentTab >= this.employees.length) {
                            this.currentTab = Math.max(0, this.employees.length - 1);
                        }
                    }
                },
                addService(empIndex) {
                    const priceVES = parseFloat(this.form.priceVES) || 0;
                    const newS = {
                        date: this.form.date,
                        client: this.form.client,
                        service: this.form.service,
                        priceVES: priceVES,
                        payment: this.form.payment,
                        commissionRate: parseFloat(this.form.commissionRate)
                    };
                    this.employees[empIndex].services.unshift(newS);
                    this.saveData();
                    this.form.client = '';
                    this.form.service = '';
                    this.form.priceVES = '';
                },
                deleteService(empIndex, sIndex) {
                    this.employees[empIndex].services.splice(sIndex, 1);
                    this.saveData();
                },
                closeWeek() {
                    if(confirm('¿Deseas cerrar la semana actual? Esto guardará los registros en el historial y dejará las cuentas en 0.')) {
                        const weekTitle = 'Semana al ' + new Date().toLocaleDateString();
                        const closedDate = new Date().toLocaleString();
                        
                        this.archives.unshift({
                            weekTitle: weekTitle,
                            closedDate: closedDate,
                            data: JSON.parse(JSON.stringify(this.employees))
                        });
                        localStorage.setItem('salon_app_archives_bs_only', JSON.stringify(this.archives));
                        
                        this.employees.forEach(emp => {
                            emp.services = [];
                        });
                        this.saveData();
                        alert('¡Semana cerrada con éxito! Cuentas reiniciadas.');
                    }
                },
                calculateTotalVES(emp) {
                    return emp.services.reduce((acc, s) => acc + s.priceVES, 0);
                },
                calculateTotalCommissionVES(emp) {
                    return emp.services.reduce((acc, s) => acc + (s.priceVES * (s.commissionRate / 100)), 0);
                },
                formatVES(val) {
                    return 'Bs. ' + val.toFixed(2);
                },
                saveData() {
                    localStorage.setItem('salon_app_data_bs_only', JSON.stringify(this.employees));
                },
                shareWhatsApp(emp) {
                    let text = `*Reporte Semanal - ${emp.name}* 💅\n\n`;
                    let totalVES = this.calculateTotalVES(emp);
                    let totalComisionVES = this.calculateTotalCommissionVES(emp);
                    
                    emp.services.forEach((s, i) => {
                        text += `${i+1}. ${s.date} | ${s.client} | ${s.service} | Bs. ${s.priceVES.toFixed(2)} [${s.payment}]\n`;
                    });
                    
                    text += `\n*Total Vendido:* Bs. ${totalVES.toFixed(2)}`;
                    text += `\n*Total Comisiones:* Bs. ${totalComisionVES.toFixed(2)}`;
                    
                    const url = `https://api.whatsapp.com/send?text=` + encodeURIComponent(text);
                    window.open(url, '_blank');
                }
            }
        }
    </script>
</body>
</html>
