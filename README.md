```react
import React, { useState, useMemo, useEffect } from 'react';
import { 
  Plus, 
  Minus, 
  Trash2, 
  Wallet, 
  TrendingUp, 
  TrendingDown, 
  PieChart, 
  List, 
  Calendar,
  Utensils,
  Car,
  ShoppingBag,
  Home,
  Coffee,
  CheckCircle2,
  X
} from 'lucide-react';

// 預設分類與圖標映射
const CATEGORIES = {
  food: { label: '餐飲', icon: Utensils, color: 'bg-orange-500' },
  transport: { label: '交通', icon: Car, color: 'bg-blue-500' },
  shopping: { label: '購物', icon: ShoppingBag, color: 'bg-pink-500' },
  housing: { label: '住房', icon: Home, color: 'bg-indigo-500' },
  entertainment: { label: '娛樂', icon: Coffee, color: 'bg-purple-500' },
  other: { label: '其他', icon: CheckCircle2, color: 'bg-gray-500' },
  income: { label: '收入', icon: TrendingUp, color: 'bg-green-500' }
};

const App = () => {
  // 狀態管理：已將初始範例資料移除，改為空陣列
  const [transactions, setTransactions] = useState([]);
  
  const [isModalOpen, setIsModalOpen] = useState(false);
  const [filterType, setFilterType] = useState('all'); // all, income, expense

  // 表單狀態
  const [formData, setFormData] = useState({
    type: 'expense',
    amount: '',
    category: 'food',
    note: '',
    date: new Date().toISOString().split('T')[0]
  });

  // 計算數值
  const stats = useMemo(() => {
    const income = transactions
      .filter(t => t.type === 'income')
      .reduce((sum, t) => sum + Number(t.amount), 0);
    const expense = transactions
      .filter(t => t.type === 'expense')
      .reduce((sum, t) => sum + Number(t.amount), 0);
    return {
      income,
      expense,
      balance: income - expense
    };
  }, [transactions]);

  // 各類別支出統計（用於簡易圖表）
  const categoryStats = useMemo(() => {
    const expensesOnly = transactions.filter(t => t.type === 'expense');
    const totals = {};
    expensesOnly.forEach(t => {
      totals[t.category] = (totals[t.category] || 0) + Number(t.amount);
    });
    return Object.entries(totals).sort((a, b) => b[1] - a[1]);
  }, [transactions]);

  // 處理表單提交
  const handleSubmit = (e) => {
    e.preventDefault();
    if (!formData.amount || formData.amount <= 0) return;

    const newTransaction = {
      ...formData,
      id: Date.now(),
      amount: Number(formData.amount)
    };

    setTransactions([newTransaction, ...transactions]);
    setIsModalOpen(false);
    setFormData({
      type: 'expense',
      amount: '',
      category: 'food',
      note: '',
      date: new Date().toISOString().split('T')[0]
    });
  };

  const deleteTransaction = (id) => {
    setTransactions(transactions.filter(t => t.id !== id));
  };

  const filteredTransactions = transactions.filter(t => 
    filterType === 'all' ? true : t.type === filterType
  );

  return (
    <div className="min-h-screen bg-gray-50 text-gray-900 pb-20">
      {/* 頂部標題 */}
      <header className="bg-white border-b sticky top-0 z-10 px-4 py-4">
        <div className="max-w-2xl mx-auto flex justify-between items-center">
          <h1 className="text-xl font-bold flex items-center gap-2">
            <Wallet className="text-blue-600" />
            <span>記帳小管家</span>
          </h1>
          <button 
            onClick={() => setIsModalOpen(true)}
            className="bg-blue-600 text-white p-2 rounded-full shadow-lg hover:bg-blue-700 transition-all active:scale-95"
          >
            <Plus size={24} />
          </button>
        </div>
      </header>

      <main className="max-w-2xl mx-auto px-4 py-6 space-y-6">
        
        {/* 資產概覽卡片 */}
        <div className="bg-gradient-to-br from-blue-600 to-indigo-700 rounded-3xl p-6 text-white shadow-xl">
          <p className="text-blue-100 text-sm font-medium">總餘額</p>
          <h2 className="text-3xl font-bold mt-1 mb-6">
            $ {stats.balance.toLocaleString()}
          </h2>
          
          <div className="grid grid-cols-2 gap-4">
            <div className="bg-white/10 rounded-2xl p-3 flex items-center gap-3">
              <div className="bg-green-400/20 p-2 rounded-lg text-green-300">
                <TrendingUp size={20} />
              </div>
              <div>
                <p className="text-xs text-blue-100">本月收入</p>
                <p className="font-bold text-lg text-green-400">${stats.income.toLocaleString()}</p>
              </div>
            </div>
            <div className="bg-white/10 rounded-2xl p-3 flex items-center gap-3">
              <div className="bg-red-400/20 p-2 rounded-lg text-red-300">
                <TrendingDown size={20} />
              </div>
              <div>
                <p className="text-xs text-blue-100">本月支出</p>
                <p className="font-bold text-lg text-red-400">${stats.expense.toLocaleString()}</p>
              </div>
            </div>
          </div>
        </div>

        {/* 支出分析圖表 */}
        {categoryStats.length > 0 ? (
          <section className="bg-white rounded-2xl p-5 shadow-sm">
            <div className="flex items-center gap-2 mb-4 font-bold text-gray-700">
              <PieChart size={18} />
              <h3>支出類別分析</h3>
            </div>
            <div className="space-y-3">
              {categoryStats.map(([cat, amount]) => {
                const percentage = ((amount / stats.expense) * 100).toFixed(1);
                const info = CATEGORIES[cat] || CATEGORIES.other;
                return (
                  <div key={cat} className="space-y-1">
                    <div className="flex justify-between text-sm">
                      <span className="text-gray-600">{info.label}</span>
                      <span className="font-medium">{percentage}% (${amount.toLocaleString()})</span>
                    </div>
                    <div className="w-full bg-gray-100 h-2 rounded-full overflow-hidden">
                      <div 
                        className={`${info.color} h-full transition-all duration-1000`} 
                        style={{ width: `${percentage}%` }}
                      ></div>
                    </div>
                  </div>
                );
              })}
            </div>
          </section>
        ) : (
          <section className="bg-white/50 border-2 border-dashed border-gray-200 rounded-2xl p-8 text-center text-gray-400">
            <div className="flex justify-center mb-2">
              <PieChart size={32} opacity={0.5} />
            </div>
            <p className="text-sm">新增第一筆支出後，這裡會顯示分析圖表</p>
          </section>
        )}

        {/* 交易列表 */}
        <section className="space-y-4">
          <div className="flex justify-between items-center">
            <h3 className="font-bold text-lg flex items-center gap-2">
              <List size={18} />
              交易紀錄
            </h3>
            <div className="flex bg-gray-200 p-1 rounded-lg text-sm">
              <button 
                onClick={() => setFilterType('all')}
                className={`px-3 py-1 rounded-md transition ${filterType === 'all' ? 'bg-white shadow text-blue-600' : 'text-gray-500'}`}
              >
                全部
              </button>
              <button 
                onClick={() => setFilterType('expense')}
                className={`px-3 py-1 rounded-md transition ${filterType === 'expense' ? 'bg-white shadow text-blue-600' : 'text-gray-500'}`}
              >
                支出
              </button>
              <button 
                onClick={() => setFilterType('income')}
                className={`px-3 py-1 rounded-md transition ${filterType === 'income' ? 'bg-white shadow text-blue-600' : 'text-gray-500'}`}
              >
                收入
              </button>
            </div>
          </div>

          <div className="space-y-3">
            {filteredTransactions.length > 0 ? (
              filteredTransactions.map(t => {
                const CategoryIcon = (CATEGORIES[t.category] || CATEGORIES.other).icon;
                const categoryColor = (CATEGORIES[t.category] || CATEGORIES.other).color;
                
                return (
                  <div key={t.id} className="bg-white p-4 rounded-2xl flex items-center gap-4 shadow-sm group hover:shadow-md transition">
                    <div className={`${categoryColor} p-3 rounded-xl text-white`}>
                      <CategoryIcon size={24} />
                    </div>
                    <div className="flex-1 min-w-0">
                      <div className="flex justify-between">
                        <h4 className="font-semibold text-gray-800 truncate">{t.note || CATEGORIES[t.category].label}</h4>
                        <p className={`font-bold ${t.type === 'income' ? 'text-green-600' : 'text-red-600'}`}>
                          {t.type === 'income' ? '+' : '-'}${t.amount.toLocaleString()}
                        </p>
                      </div>
                      <div className="flex justify-between items-center text-xs text-gray-400 mt-1">
                        <div className="flex items-center gap-1">
                          <Calendar size={12} />
                          {t.date}
                        </div>
                        <button 
                          onClick={() => deleteTransaction(t.id)}
                          className="text-gray-300 hover:text-red-500 transition opacity-0 group-hover:opacity-100"
                        >
                          <Trash2 size={14} />
                        </button>
                      </div>
                    </div>
                  </div>
                );
              })
            ) : (
              <div className="text-center py-12 text-gray-400 bg-white rounded-2xl border border-gray-100">
                目前尚無交易紀錄
              </div>
            )}
          </div>
        </section>
      </main>

      {/* 新增紀錄的彈出視窗 */}
      {isModalOpen && (
        <div className="fixed inset-0 z-50 flex items-end sm:items-center justify-center p-4">
          <div className="absolute inset-0 bg-black/60 backdrop-blur-sm" onClick={() => setIsModalOpen(false)}></div>
          <div className="bg-white w-full max-w-lg rounded-t-3xl sm:rounded-3xl p-6 relative z-10 animate-in slide-in-from-bottom duration-300">
            <div className="flex justify-between items-center mb-6">
              <h2 className="text-xl font-bold">新增記帳</h2>
              <button onClick={() => setIsModalOpen(false)} className="p-2 hover:bg-gray-100 rounded-full">
                <X size={24} className="text-gray-500" />
              </button>
            </div>

            <form onSubmit={handleSubmit} className="space-y-4">
              <div className="flex p-1 bg-gray-100 rounded-xl">
                <button
                  type="button"
                  onClick={() => setFormData({...formData, type: 'expense', category: 'food'})}
                  className={`flex-1 py-2 rounded-lg font-medium transition ${formData.type === 'expense' ? 'bg-red-500 text-white shadow-lg' : 'text-gray-500'}`}
                >
                  支出
                </button>
                <button
                  type="button"
                  onClick={() => setFormData({...formData, type: 'income', category: 'income'})}
                  className={`flex-1 py-2 rounded-lg font-medium transition ${formData.type === 'income' ? 'bg-green-500 text-white shadow-lg' : 'text-gray-500'}`}
                >
                  收入
                </button>
              </div>

              <div>
                <label className="block text-sm font-medium text-gray-700 mb-1">金額</label>
                <div className="relative">
                  <span className="absolute left-4 top-1/2 -translate-y-1/2 text-xl font-bold text-gray-400">$</span>
                  <input
                    type="number"
                    required
                    placeholder="0"
                    value={formData.amount}
                    onChange={(e) => setFormData({...formData, amount: e.target.value})}
                    className="w-full bg-gray-50 border-none rounded-2xl py-4 pl-10 pr-4 text-2xl font-bold focus:ring-2 focus:ring-blue-500 transition"
                  />
                </div>
              </div>

              {formData.type === 'expense' && (
                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-2">類別</label>
                  <div className="grid grid-cols-3 gap-2">
                    {Object.entries(CATEGORIES).filter(([k]) => k !== 'income').map(([key, info]) => (
                      <button
                        key={key}
                        type="button"
                        onClick={() => setFormData({...formData, category: key})}
                        className={`flex flex-col items-center gap-1 p-3 rounded-2xl border-2 transition ${formData.category === key ? 'border-blue-500 bg-blue-50 text-blue-600' : 'border-transparent bg-gray-50 text-gray-500'}`}
                      >
                        <info.icon size={20} />
                        <span className="text-xs font-medium">{info.label}</span>
                      </button>
                    ))}
                  </div>
                </div>
              )}

              <div className="grid grid-cols-2 gap-4">
                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">日期</label>
                  <input
                    type="date"
                    value={formData.date}
                    onChange={(e) => setFormData({...formData, date: e.target.value})}
                    className="w-full bg-gray-50 border-none rounded-xl p-3 text-sm focus:ring-2 focus:ring-blue-500 transition"
                  />
                </div>
                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">備註</label>
                  <input
                    type="text"
                    placeholder="點擊輸入..."
                    value={formData.note}
                    onChange={(e) => setFormData({...formData, note: e.target.value})}
                    className="w-full bg-gray-50 border-none rounded-xl p-3 text-sm focus:ring-2 focus:ring-blue-500 transition"
                  />
                </div>
              </div>

              <button
                type="submit"
                className={`w-full py-4 rounded-2xl font-bold text-white shadow-xl mt-4 transition-all active:scale-95 ${formData.type === 'income' ? 'bg-green-600 hover:bg-green-700' : 'bg-red-600 hover:bg-red-700'}`}
              >
                儲存紀錄
              </button>
            </form>
          </div>
        </div>
      )}
    </div>
  );
};

export default App;

```
