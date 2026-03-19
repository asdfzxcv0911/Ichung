import React, { useState, useMemo } from 'react';
import { 
  ShoppingBag, 
  Package, 
  History, 
  User, 
  ChevronRight, 
  Plus, 
  Minus, 
  Trash2, 
  CheckCircle2, 
  PhoneCall,
  Search,
  Filter,
  Car,
  Calendar,
  AlertCircle,
  Truck,
  Building2,
  MapPin,
  Save,
  Lock,
  Unlock
} from 'lucide-react';

// 產品數據
const PRODUCTS = [
  // TOYOTA
  { id: 1, brand: 'Toyota', model: 'Altis', years: ['2001-2007 (E120)', '2008-2013 (E140)', '2014-2018 (E170)', '2019-至今 (E210)'], name: 'Toyota Altis 鋁製散熱水箱', price: 1800, image: 'https://images.unsplash.com/photo-1486006920555-c77dcf18193c?w=200&h=200&fit=crop', stock: 25 },
  { id: 2, brand: 'Toyota', model: 'Camry', years: ['2002-2006 (3.0)', '2006-2011 (2.0/2.4)', '2012-2018 (Hybrid)', '2019-至今'], name: 'Toyota Camry 強化型水箱', price: 2800, image: 'https://images.unsplash.com/photo-1486006920555-c77dcf18193c?w=200&h=200&fit=crop', stock: 12 },
  // HONDA
  { id: 10, brand: 'Honda', model: 'Civic', years: ['1996-2000 (K8)', '2001-2005 (K10/Ferio)', '2006-2011 (K12)', '2012-2016 (K14)'], name: 'Honda Civic 高性能水箱', price: 2600, image: 'https://images.unsplash.com/photo-1486006920555-c77dcf18193c?w=200&h=200&fit=crop', stock: 18 },
  // MITSUBISHI
  { id: 31, brand: 'Mitsubishi', model: 'Veryca (菱利)', years: ['2000-2018 (1.2/1.3)', '2018-至今 (A180/A190)'], name: 'Mitsubishi 菱利 A180/A190 強化型水箱', price: 1500, image: 'https://images.unsplash.com/photo-1486006920555-c77dcf18193c?w=200&h=200&fit=crop', stock: 45 },
];

const App = () => {
  const [activeTab, setActiveTab] = useState('home');
  const [cart, setCart] = useState([]);
  const [orders, setOrders] = useState([
    { id: 'ORD-20240501', date: '2024-05-10', total: 5400, status: '已完成' }
  ]);

  // 客戶資訊狀態
  const [customerInfo, setCustomerInfo] = useState({
    companyName: '',
    contactPerson: '',
    phone: '',
    address: ''
  });
  
  // 是否已解鎖下單功能
  const [isUnlocked, setIsUnlocked] = useState(false);
  const [showUnlockAnim, setShowUnlockAnim] = useState(false);

  const [filterBrand, setFilterBrand] = useState('');
  const [filterModel, setFilterModel] = useState('');
  const [filterYear, setFilterYear] = useState('');
  const [showOrderSuccess, setShowOrderSuccess] = useState(false);
  const [pendingQuantities, setPendingQuantities] = useState({});

  const brands = useMemo(() => [...new Set(PRODUCTS.map(p => p.brand))].sort(), []);

  const models = useMemo(() => {
    if (!filterBrand) return [];
    return [...new Set(PRODUCTS.filter(p => p.brand === filterBrand).map(p => p.model))].sort();
  }, [filterBrand]);

  const years = useMemo(() => {
    if (!filterModel) return [];
    return PRODUCTS.filter(p => p.brand === filterBrand && p.model === filterModel).flatMap(p => p.years);
  }, [filterBrand, filterModel]);

  const searchResults = useMemo(() => {
    if (!filterBrand || !filterModel || !filterYear) return [];
    return PRODUCTS.filter(p => p.brand === filterBrand && p.model === filterModel && p.years.includes(filterYear));
  }, [filterBrand, filterModel, filterYear]);

  const handleSaveProfile = () => {
    const { companyName, contactPerson, phone, address } = customerInfo;
    if (companyName && contactPerson && phone && address) {
      setShowUnlockAnim(true);
      setTimeout(() => {
        setIsUnlocked(true);
        setShowUnlockAnim(false);
      }, 1500);
    } else {
      const form = document.getElementById('profile-form');
      if (form) {
        form.classList.add('animate-shake');
        setTimeout(() => form.classList.remove('animate-shake'), 500);
      }
    }
  };

  const addToCart = (product, quantity) => {
    if (!isUnlocked) return;
    const existing = cart.find(item => item.id === product.id);
    if (existing) {
      setCart(cart.map(item => item.id === product.id ? { ...item, quantity: item.quantity + quantity, selectedYear: filterYear } : item));
    } else {
      setCart([...cart, { ...product, quantity: quantity, selectedYear: filterYear }]);
    }
    setPendingQuantities(prev => ({ ...prev, [product.id]: 1 }));
  };

  const updateQuantity = (id, delta) => {
    setCart(cart.map(item => {
      if (item.id === id) {
        return { ...item, quantity: Math.max(1, item.quantity + delta) };
      }
      return item;
    }));
  };

  const handlePendingQtyChange = (productId, delta) => {
    const currentQty = pendingQuantities[productId] || 1;
    setPendingQuantities({
      ...pendingQuantities,
      [productId]: Math.max(1, currentQty + delta)
    });
  };

  const handleCheckout = () => {
    if (cart.length === 0) return;
    const total = cart.reduce((sum, item) => sum + (item.price * item.quantity), 0);
    const newOrder = {
      id: `ORD-${Date.now().toString().slice(-8)}`,
      date: new Date().toISOString().split('T')[0],
      total,
      status: '新訂單'
    };
    setOrders([newOrder, ...orders]);
    setCart([]);
    setShowOrderSuccess(true);
    setTimeout(() => {
      setShowOrderSuccess(false);
      setActiveTab('orders');
    }, 2000);
  };

  const renderHome = () => (
    <div className="p-4 space-y-6 animate-in fade-in duration-500">
      <div className="bg-gradient-to-br from-blue-700 to-blue-500 text-white p-10 rounded-[2.5rem] shadow-lg relative overflow-hidden">
        <Car className="absolute -bottom-6 -right-6 text-white/10 w-40 h-40 rotate-12" />
      </div>

      <div id="profile-form" className={`bg-white p-6 rounded-[2.5rem] shadow-sm border ${isUnlocked ? 'border-green-100' : 'border-blue-100'} transition-all duration-500`}>
        <div className="flex items-center justify-between mb-6">
          <div className="flex items-center gap-3">
            <div className={`w-10 h-10 ${isUnlocked ? 'bg-green-500' : 'bg-blue-600'} text-white rounded-2xl flex items-center justify-center font-black shadow-lg shadow-blue-100 transition-colors`}>
              {isUnlocked ? <CheckCircle2 size={20} /> : "1"}
            </div>
            <div>
              <h3 className="font-black text-gray-800">公司基本資訊</h3>
              <p className="text-[10px] text-gray-400 font-bold uppercase tracking-wider">Required Profile</p>
            </div>
          </div>
          {!isUnlocked && <Lock size={16} className="text-blue-200" />}
        </div>
        
        <div className="space-y-4">
          <div className="space-y-1.5">
            <label className="text-[11px] font-black text-gray-400 uppercase tracking-widest ml-1">公司名稱 <span className="text-red-500">*</span></label>
            <div className="relative">
              <Building2 className="absolute left-4 top-1/2 -translate-y-1/2 text-gray-300" size={18} />
              <input disabled={isUnlocked} type="text" placeholder="請輸入公司完整抬頭" className={`w-full pl-12 pr-4 py-4 rounded-2xl text-sm font-bold outline-none transition-all border-2 ${isUnlocked ? 'bg-gray-50 border-transparent text-gray-500' : 'bg-white border-gray-50 focus:border-blue-500 focus:shadow-md'}`} value={customerInfo.companyName} onChange={(e) => setCustomerInfo({...customerInfo, companyName: e.target.value})} />
            </div>
          </div>
          <div className="grid grid-cols-2 gap-4">
            <div className="space-y-1.5">
              <label className="text-[11px] font-black text-gray-400 uppercase tracking-widest ml-1">聯絡人 <span className="text-red-500">*</span></label>
              <input disabled={isUnlocked} type="text" placeholder="姓名" className={`w-full px-5 py-4 rounded-2xl text-sm font-bold outline-none transition-all border-2 ${isUnlocked ? 'bg-gray-50 border-transparent text-gray-500' : 'bg-white border-gray-50 focus:border-blue-500'}`} value={customerInfo.contactPerson} onChange={(e) => setCustomerInfo({...customerInfo, contactPerson: e.target.value})} />
            </div>
            <div className="space-y-1.5">
              <label className="text-[11px] font-black text-gray-400 uppercase tracking-widest ml-1">聯絡電話 <span className="text-red-500">*</span></label>
              <input disabled={isUnlocked} type="tel" placeholder="手機/市話" className={`w-full px-5 py-4 rounded-2xl text-sm font-bold outline-none transition-all border-2 ${isUnlocked ? 'bg-gray-50 border-transparent text-gray-500' : 'bg-white border-gray-50 focus:border-blue-500'}`} value={customerInfo.phone} onChange={(e) => setCustomerInfo({...customerInfo, phone: e.target.value})} />
            </div>
          </div>
          <div className="space-y-1.5">
            <div className="flex justify-between items-center ml-1">
              <label className="text-[11px] font-black text-gray-400 uppercase tracking-widest">配送地址 <span className="text-red-500">*</span></label>
              {!isUnlocked && <span className="text-[9px] bg-red-100 text-red-600 px-2 py-0.5 rounded-lg font-black italic">新客戶必填項目</span>}
            </div>
            <div className="relative">
              <MapPin className="absolute left-4 top-1/2 -translate-y-1/2 text-gray-300" size={18} />
              <input disabled={isUnlocked} type="text" placeholder="完整配送地址（含行政區）" className={`w-full pl-12 pr-4 py-4 rounded-2xl text-sm font-bold outline-none transition-all border-2 ${isUnlocked ? 'bg-gray-50 border-transparent text-gray-500' : 'bg-white border-gray-50 focus:border-blue-500'}`} value={customerInfo.address} onChange={(e) => setCustomerInfo({...customerInfo, address: e.target.value})} />
            </div>
          </div>
        </div>

        {!isUnlocked ? (
          <button onClick={handleSaveProfile} className="w-full bg-blue-600 text-white py-5 rounded-3xl font-black text-base flex items-center justify-center gap-3 shadow-xl shadow-blue-100 active:scale-95 transition-all mt-8">
            <Save size={20} /> 儲存資訊並解鎖進貨功能
          </button>
        ) : (
          <div className="mt-6 flex items-center justify-center gap-2 py-2 text-green-600 font-black text-sm">
            <Unlock size={16} /> 帳號已驗證成功，可開始下單
          </div>
        )}
      </div>

      <div className={`space-y-6 transition-all duration-500 ${!isUnlocked ? 'opacity-40 grayscale pointer-events-none' : 'opacity-100 grayscale-0'}`}>
        <div className="space-y-3 px-2">
          <h3 className="font-black text-gray-800 flex items-center gap-2">快速捷徑 {!isUnlocked && <Lock size={14} className="text-gray-400" />}</h3>
          <button onClick={() => setActiveTab('products')} className="w-full bg-white p-6 rounded-[2.5rem] shadow-sm border border-gray-50 flex items-center justify-between group active:bg-blue-50">
            <div className="flex items-center gap-5">
              <div className="w-14 h-14 bg-blue-600 text-white rounded-[1.5rem] flex items-center justify-center shadow-xl shadow-blue-100 group-hover:rotate-12 transition-transform">
                <Search size={28} />
              </div>
              <div className="text-left">
                <div className="font-black text-gray-800 text-lg">立即找水箱</div>
                <p className="text-[10px] text-gray-400 font-bold uppercase tracking-widest mt-1">Ready to Order</p>
              </div>
            </div>
            <ChevronRight className="text-gray-200" />
          </button>
        </div>

        <div className="grid grid-cols-2 gap-4">
          <div className="bg-white p-6 rounded-[2.5rem] shadow-sm border border-gray-50">
            <div className="w-12 h-12 bg-green-50 text-green-600 rounded-2xl flex items-center justify-center mb-4"><Truck size={24} /></div>
            <div className="text-[10px] text-gray-400 font-black uppercase tracking-tighter">本月次數</div>
            <div className="font-black text-2xl text-gray-800 mt-1 tracking-tighter">0 <span className="text-xs">次</span></div>
          </div>
          <div className="bg-white p-6 rounded-[2.5rem] shadow-sm border border-gray-50">
            <div className="w-12 h-12 bg-orange-50 text-orange-600 rounded-2xl flex items-center justify-center mb-4"><Package size={24} /></div>
            <div className="text-[10px] text-gray-400 font-black uppercase tracking-tighter">帳戶等級</div>
            <div className="font-black text-2xl text-orange-600 mt-1">一般會員</div>
          </div>
        </div>
      </div>
      
      <div className="bg-blue-600 text-white p-6 rounded-[2.5rem] shadow-xl shadow-blue-100 flex items-center justify-between">
        <div className="flex items-center gap-4">
          <div className="p-3 bg-white/20 rounded-2xl backdrop-blur-md"><PhoneCall size={24} /></div>
          <div>
            <div className="text-[10px] text-white/70 font-black tracking-widest uppercase">業務諮詢專線</div>
            <div className="font-black text-xl tracking-tighter">07-XXX-XXXX</div>
          </div>
        </div>
        <button className="bg-white/10 p-2 rounded-full"><ChevronRight size={20} /></button>
      </div>
    </div>
  );

  const renderProducts = () => (
    <div className="p-4 pb-24 animate-in slide-in-from-right duration-500">
      <div className="mb-6 bg-white p-6 rounded-[2.5rem] shadow-sm space-y-6 border border-gray-50">
        <div className="flex items-center gap-2 text-blue-800"><Filter size={20} /><span className="font-black text-xl tracking-tight">找特定水箱</span></div>
        <div className="space-y-5">
          <div className="space-y-1.5">
            <label className="text-[11px] font-black text-gray-400 uppercase tracking-[0.2em] ml-1">汽車品牌</label>
            <select className="w-full p-4 bg-gray-50 rounded-2xl border-2 border-transparent focus:border-blue-500 focus:bg-white transition-all text-sm font-bold outline-none" value={filterBrand} onChange={(e) => { setFilterBrand(e.target.value); setFilterModel(''); setFilterYear(''); }}>
              <option value="">選擇品牌</option>
              {brands.map(b => <option key={b} value={b}>{b}</option>)}
            </select>
          </div>
          <div className="space-y-1.5">
            <label className="text-[11px] font-black text-gray-400 uppercase tracking-[0.2em] ml-1">車型名稱</label>
            <select disabled={!filterBrand} className="w-full p-4 bg-gray-50 rounded-2xl border-2 border-transparent focus:border-blue-500 focus:bg-white transition-all text-sm font-bold disabled:opacity-40 outline-none" value={filterModel} onChange={(e) => { setFilterModel(e.target.value); setFilterYear(''); }}>
              <option value="">選擇型號</option>
              {models.map(m => <option key={m} value={m}>{m}</option>)}
            </select>
          </div>
          <div className="space-y-1.5">
            <label className="text-[11px] font-black text-gray-400 uppercase tracking-[0.2em] ml-1">生產年份</label>
            <select disabled={!filterModel} className="w-full p-4 bg-gray-50 rounded-2xl border-2 border-transparent focus:border-blue-500 focus:bg-white transition-all text-sm font-bold disabled:opacity-40 outline-none" value={filterYear} onChange={(e) => setFilterYear(e.target.value)}>
              <option value="">選擇年份區間</option>
              {years.map(y => <option key={y} value={y}>{y}</option>)}
            </select>
          </div>
        </div>
      </div>

      <div className="space-y-4">
        {searchResults.length > 0 ? (
          searchResults.map(product => {
            const currentPendingQty = pendingQuantities[product.id] || 1;
            return (
              <div key={product.id} className="bg-white p-5 rounded-[2rem] shadow-sm flex gap-5 border border-transparent hover:border-blue-200 active:scale-[0.98] transition-all">
                <div className="w-24 h-24 bg-gray-100 rounded-3xl overflow-hidden shrink-0"><img src={product.image} alt={product.name} className="w-full h-full object-cover" /></div>
                <div className="flex-1 flex flex-col justify-between py-1">
                  <div>
                    <h4 className="font-black text-gray-800 text-sm leading-tight line-clamp-2">{product.name}</h4>
                    <p className="text-red-500 font-black text-lg mt-1 tracking-tighter">${product.price.toLocaleString()}</p>
                  </div>
                  <div className="flex items-center justify-between mt-3">
                    <div className="flex items-center gap-2 bg-gray-50 p-1.5 rounded-xl border border-gray-100">
                      <button onClick={() => handlePendingQtyChange(product.id, -1)} className="p-1 rounded-lg bg-white shadow-sm"><Minus size={12} /></button>
                      <span className="w-6 text-center font-black text-xs">{currentPendingQty}</span>
                      <button onClick={() => handlePendingQtyChange(product.id, 1)} className="p-1 rounded-lg bg-white shadow-sm"><Plus size={12} /></button>
                    </div>
                    <button onClick={() => addToCart(product, currentPendingQty)} className="bg-blue-600 text-white px-4 py-2.5 rounded-xl font-black text-xs flex items-center gap-1.5 shadow-lg shadow-blue-100 active:bg-blue-800 transition-colors"><Plus size={14} /> 加入清單</button>
                  </div>
                </div>
              </div>
            );
          })
        ) : (
          <div className="text-center py-20 px-10 bg-white rounded-[2.5rem] border border-dashed border-gray-200"><Search size={40} className="mx-auto text-gray-200 mb-4" /><p className="text-gray-400 font-bold text-sm">請完成篩選以顯示產品</p></div>
        )}
      </div>
    </div>
  );

  const renderCart = () => {
    const total = cart.reduce((sum, item) => sum + (item.price * item.quantity), 0);
    return (
      <div className="p-4 pb-24 animate-in slide-in-from-bottom duration-500">
        <h3 className="text-2xl font-black mb-6 flex items-center gap-3 px-2"><ShoppingBag className="text-blue-600" /> 待訂購清單</h3>
        {cart.length === 0 ? (
          <div className="text-center py-24 text-gray-300"><div className="w-20 h-20 bg-gray-50 rounded-full flex items-center justify-center mx-auto mb-6"><ShoppingBag size={40} className="opacity-20" /></div><p className="font-black text-gray-400">目前沒有選購任何商品</p><button onClick={() => setActiveTab('products')} className="mt-6 bg-blue-600 text-white px-8 py-3 rounded-2xl font-black text-sm shadow-xl shadow-blue-100 transition-all active:scale-95">立即找貨</button></div>
        ) : (
          <div className="space-y-4">
            {cart.map(item => (
              <div key={item.id} className="bg-white p-5 rounded-[2rem] shadow-sm flex items-center gap-5">
                <img src={item.image} alt={item.name} className="w-16 h-16 object-cover rounded-2xl shrink-0" />
                <div className="flex-1"><h4 className="font-black text-xs text-gray-800 leading-tight">{item.name}</h4><p className="text-red-500 font-black text-sm mt-1 tracking-tighter">${item.price.toLocaleString()}</p></div>
                <div className="flex items-center gap-2 bg-gray-50 p-2 rounded-2xl border border-gray-100">
                  <button onClick={() => updateQuantity(item.id, -1)} className="p-1.5 rounded-xl bg-white shadow-sm"><Minus size={12} /></button>
                  <span className="w-6 text-center font-black text-sm">{item.quantity}</span>
                  <button onClick={() => updateQuantity(item.id, 1)} className="p-1.5 rounded-xl bg-white shadow-sm"><Plus size={12} /></button>
                  <button onClick={() => setCart(cart.filter(i => i.id !== item.id))} className="ml-2 text-gray-300 hover:text-red-500"><Trash2 size={16} /></button>
                </div>
              </div>
            ))}
            <div className="bg-white p-8 rounded-[2.5rem] shadow-sm mt-8 space-y-5 border border-gray-50">
              <div className="flex justify-between items-center"><span className="text-xs font-black text-gray-400 uppercase tracking-widest">總金額預估</span><span className="text-3xl font-black text-red-600 tracking-tighter">${total.toLocaleString()}</span></div>
              <button onClick={handleCheckout} className="w-full bg-blue-600 text-white py-5 rounded-3xl font-black text-lg shadow-2xl shadow-blue-100 active:scale-95 transition-all">送出進貨訂單</button>
            </div>
          </div>
        )}
      </div>
    );
  };

  const renderOrders = () => (
    <div className="p-4 pb-24 animate-in slide-in-from-left duration-500">
      <h3 className="text-2xl font-black mb-6 px-2">進貨歷史紀錄</h3>
      <div className="space-y-4">
        {orders.map(order => (
          <div key={order.id} className="bg-white p-6 rounded-[2.5rem] shadow-sm border-l-[6px] border-blue-600 transition-all hover:shadow-md">
            <div className="flex justify-between items-start mb-4">
              <div><div className="font-black text-gray-800 tracking-tight text-lg">{order.id}</div><div className="text-[10px] text-gray-400 font-bold uppercase tracking-widest mt-1">{order.date}</div></div>
              <span className={`px-4 py-1.5 rounded-full text-[10px] font-black uppercase tracking-widest ${order.status === '已完成' ? 'bg-green-100 text-green-600' : 'bg-orange-100 text-orange-600'}`}>{order.status}</span>
            </div>
            <div className="flex justify-between items-center pt-4 border-t border-gray-50"><span className="text-lg font-black text-gray-700 tracking-tighter">NT$ {order.total.toLocaleString()}</span><button className="text-blue-600 text-xs font-black flex items-center gap-1 group">詳細內容 <ChevronRight size={14} className="group-hover:translate-x-1 transition-transform" /></button></div>
          </div>
        ))}
      </div>
    </div>
  );

  return (
    <div className="flex flex-col h-screen bg-gray-50 max-w-md mx-auto relative overflow-hidden font-sans text-gray-900 shadow-2xl border-x border-gray-200">
      <header className="bg-white/80 backdrop-blur-xl px-6 py-5 flex items-center justify-between shadow-sm z-30 sticky top-0">
        <div className="flex items-center gap-3">
          <div className="w-10 h-10 bg-blue-600 rounded-2xl flex items-center justify-center text-white font-black shadow-blue-200 shadow-xl italic text-xl">乙</div>
          <div><h1 className="font-black text-xl text-gray-800 leading-none tracking-tight">乙昌水箱行</h1><div className="flex items-center gap-1 mt-1.5"><span className="text-[8px] bg-blue-50 text-blue-600 px-2 py-0.5 rounded-full font-black tracking-widest uppercase">B2B Portal</span></div></div>
        </div>
        <div className="flex items-center gap-3"><div className={`p-2 rounded-xl ${isUnlocked ? 'text-green-500 bg-green-50' : 'text-gray-300 bg-gray-50'}`}>{isUnlocked ? <Unlock size={20} /> : <Lock size={20} />}</div></div>
      </header>

      <main className="flex-1 overflow-y-auto">
        {activeTab === 'home' && renderHome()}
        {activeTab === 'products' && renderProducts()}
        {activeTab === 'cart' && renderCart()}
        {activeTab === 'orders' && renderOrders()}
      </main>

      {showUnlockAnim && (
        <div className="absolute inset-0 z-50 flex flex-col items-center justify-center bg-blue-600 text-white animate-in fade-in duration-300">
          <div className="animate-bounce mb-6 bg-white/20 p-8 rounded-full"><Unlock size={80} /></div>
          <h2 className="text-3xl font-black">驗證成功！</h2><p className="mt-2 font-bold opacity-80">正在為您開啟訂貨系統...</p>
        </div>
      )}

      {showOrderSuccess && (
        <div className="absolute inset-0 z-50 flex items-center justify-center bg-gray-900/60 backdrop-blur-md px-10 animate-in fade-in duration-300">
          <div className="bg-white p-12 rounded-[3.5rem] text-center shadow-2xl scale-110 border-4 border-white">
            <div className="w-24 h-24 bg-green-50 text-green-500 rounded-[2rem] flex items-center justify-center mx-auto mb-6 animate-bounce shadow-inner"><CheckCircle2 size={56} /></div>
            <h3 className="text-2xl font-black text-gray-800 tracking-tight">訂單已送出</h3><p className="text-gray-400 text-sm mt-3 font-medium px-4">乙昌物流小組正準備為您出貨。</p>
          </div>
        </div>
      )}

      <nav className="bg-white/95 backdrop-blur-2xl border-t border-gray-100 py-5 px-10 flex justify-between items-center z-30 shadow-[0_-15px_40px_rgba(0,0,0,0.04)]">
        <button onClick={() => setActiveTab('home')} className={`flex flex-col items-center gap-1.5 transition-all ${activeTab === 'home' ? 'text-blue-600 scale-110' : 'text-gray-300'}`}><Package size={24} strokeWidth={activeTab === 'home' ? 2.5 : 2} /><span className="text-[10px] font-black uppercase tracking-tighter">首頁</span></button>
        <button onClick={() => isUnlocked && setActiveTab('products')} className={`flex flex-col items-center gap-1.5 transition-all relative ${activeTab === 'products' ? 'text-blue-600 scale-110' : 'text-gray-300'} ${!isUnlocked && 'opacity-30 cursor-not-allowed'}`}><Search size={24} strokeWidth={activeTab === 'products' ? 2.5 : 2} />{!isUnlocked && <Lock className="absolute -top-1 -right-1 text-gray-400" size={10} />}<span className="text-[10px] font-black uppercase tracking-tighter">找水箱</span></button>
        <button onClick={() => isUnlocked && setActiveTab('cart')} className={`flex flex-col items-center gap-1.5 relative transition-all ${activeTab === 'cart' ? 'text-blue-600 scale-110' : 'text-gray-300'} ${!isUnlocked && 'opacity-30 cursor-not-allowed'}`}><div className="relative"><ShoppingBag size={24} strokeWidth={activeTab === 'cart' ? 2.5 : 2} />{cart.length > 0 && <span className="absolute -top-2 -right-2 bg-red-500 text-white text-[9px] w-4 h-4 rounded-full flex items-center justify-center font-black border-2 border-white shadow-lg">{cart.length}</span>}</div>{!isUnlocked && <Lock className="absolute -top-1 -right-1 text-gray-400" size={10} />}<span className="text-[10px] font-black uppercase tracking-tighter">清單</span></button>
        <button onClick={() => isUnlocked && setActiveTab('orders')} className={`flex flex-col items-center gap-1.5 transition-all ${activeTab === 'orders' ? 'text-blue-600 scale-110' : 'text-gray-300'} ${!isUnlocked && 'opacity-30 cursor-not-allowed'}`}><History size={24} strokeWidth={activeTab === 'orders' ? 2.5 : 2} />{!isUnlocked && <Lock className="absolute -top-1 -right-1 text-gray-400" size={10} />}<span className="text-[10px] font-black uppercase tracking-tighter">紀錄</span></button>
      </nav>
      
      <style>{`
        .animate-shake { animation: shake 0.5s cubic-bezier(.36,.07,.19,.97) both; }
        @keyframes shake { 10%, 90% { transform: translate3d(-1px, 0, 0); } 20%, 80% { transform: translate3d(2px, 0, 0); } 30%, 50%, 70% { transform: translate3d(-4px, 0, 0); } 40%, 60% { transform: translate3d(4px, 0, 0); } }
      `}</style>
    </div>
  );
};

export default App;
