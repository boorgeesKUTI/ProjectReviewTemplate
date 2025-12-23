import React, { useState, useMemo } from 'react';
import { 
  PieChart, Pie, Cell, BarChart, Bar, XAxis, YAxis, CartesianGrid, Tooltip as RechartsTooltip, Legend, ResponsiveContainer, LineChart, Line 
} from 'recharts';
import { 
  LayoutDashboard, ListTodo, Bug, Lightbulb, TrendingUp, Calendar, 
  Plus, Trash2, ArrowRight, ChevronRight, Save, Download, Wrench, Zap,
  Search, History, X, Clock, CheckCircle2
} from 'lucide-react';

// --- Mock Data / Initial State Structure ---

const INITIAL_PROJECTS = [
  { 
    id: 1, 
    name: "示例：A公司CRM重构", 
    client: "A科技", 
    industry: "SaaS", 
    startDate: "2025-01-15", 
    endDate: "2025-03-30", 
    plannedDays: 45, 
    actualDays: 50, 
    status: "已上线", 
    type: "重构",
    skills: "微前端架构, React Hooks",
    tools: "Figma, Jira, Notion",
    milestones: [
        { date: "2025-01-15", title: "项目启动", type: "start" },
        { date: "2025-02-10", title: "完成核心模块开发", type: "milestone" },
        { date: "2025-03-25", title: "SIT测试通过", type: "milestone" },
        { date: "2025-03-30", title: "正式上线", type: "end" }
    ]
  },
  { 
    id: 2, 
    name: "示例：B银行活动页", 
    client: "B银行", 
    industry: "金融", 
    startDate: "2025-04-01", 
    endDate: "2025-04-20", 
    plannedDays: 15, 
    actualDays: 12, 
    status: "已上线", 
    type: "H5",
    skills: "Canvas动画, 性能优化",
    tools: "Photoshop, VSCode",
    milestones: [
        { date: "2025-04-01", title: "需求确认", type: "start" },
        { date: "2025-04-20", title: "上线交付", type: "end" }
    ]
  },
];

const INITIAL_BUGS = [
  { id: 1, project: "A公司CRM重构", issue: "上线后高并发导致接口超时", cause: "缺少压力测试", solution: "增加Redis缓存层", tag: "技术债" },
];

const INITIAL_HIGHLIGHTS = [
  { id: 1, category: "工具", title: "Figma 变量功能", content: "使用Variable管理设计系统，交付效率提升30%" },
  { id: 2, category: "沟通", title: "每日站会改良", content: "改为只同步'障碍'，会议时间从30分钟缩短至10分钟" },
];

const COLORS = ['#0088FE', '#00C49F', '#FFBB28', '#FF8042', '#8884d8', '#82ca9d'];
const TOOL_COLORS = ['#8884d8', '#82ca9d', '#ffc658', '#ff7300', '#0088fe'];

// --- Components ---

const Card = ({ title, children, className = "" }) => (
  <div className={`bg-white p-6 rounded-xl shadow-sm border border-gray-100 ${className}`}>
    <h3 className="text-lg font-semibold text-gray-800 mb-4 flex items-center">{title}</h3>
    {children}
  </div>
);

const StatCard = ({ title, value, subtext, trend }) => (
  <div className="bg-white p-6 rounded-xl shadow-sm border border-gray-100">
    <div className="text-gray-500 text-sm font-medium mb-1">{title}</div>
    <div className="text-3xl font-bold text-gray-800 mb-2">{value}</div>
    <div className={`text-xs flex items-center ${trend === 'up' ? 'text-green-500' : trend === 'down' ? 'text-red-500' : 'text-gray-400'}`}>
      {subtext}
    </div>
  </div>
);

// --- Sub-View Components ---

const DashboardView = ({ stats, projects }) => (
  <div className="space-y-6 animate-in fade-in duration-500">
    {/* KPI Row */}
    <div className="grid grid-cols-1 md:grid-cols-4 gap-4">
      <StatCard title="年度项目总数" value={stats.totalProjects} subtext="较去年 +10% (模拟)" trend="up" />
      <StatCard title="平均工期偏差率" value={`${Math.abs(100 - stats.efficiencyRate)}%`} subtext={stats.efficiencyRate >= 100 ? "效率超前" : "存在延期"} trend={stats.efficiencyRate >= 100 ? "up" : "down"} />
      <StatCard title="累计投入工时(天)" value={stats.totalActualDays} subtext="本年度个人净投入" trend="neutral" />
      <StatCard title="新掌握技能点" value={stats.uniqueSkills.length} subtext="基于项目履历统计" trend="up" />
    </div>

    {/* Charts Row 1 */}
    <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
      <Card title="项目交付节奏与效率趋势">
          <div className="h-64">
              <ResponsiveContainer width="100%" height="100%">
                  <LineChart data={stats.monthlyData}>
                      <CartesianGrid strokeDasharray="3 3" vertical={false} />
                      <XAxis dataKey="name" axisLine={false} tickLine={false} />
                      <YAxis yAxisId="left" axisLine={false} tickLine={false} />
                      <YAxis yAxisId="right" orientation="right" axisLine={false} tickLine={false} unit="%" />
                      <RechartsTooltip />
                      <Legend />
                      <Line yAxisId="left" type="monotone" dataKey="count" name="交付数量" stroke="#8884d8" strokeWidth={3} />
                      <Line yAxisId="right" type="monotone" dataKey="efficiency" name="效率指数" stroke="#82ca9d" strokeWidth={2} strokeDasharray="5 5" />
                  </LineChart>
              </ResponsiveContainer>
          </div>
          <p className="text-xs text-gray-400 mt-2 text-center">* 效率指数 &gt; 100% 代表实际工期短于预期</p>
      </Card>

      <Card title="服务行业分布">
           <div className="h-64">
              <ResponsiveContainer width="100%" height="100%">
                  <PieChart>
                      <Pie
                          data={stats.industryData}
                          cx="50%"
                          cy="50%"
                          innerRadius={60}
                          outerRadius={80}
                          fill="#8884d8"
                          paddingAngle={5}
                          dataKey="value"
                          label
                      >
                          {stats.industryData.map((entry, index) => (
                              <Cell key={`cell-${index}`} fill={COLORS[index % COLORS.length]} />
                          ))}
                      </Pie>
                      <RechartsTooltip />
                      <Legend />
                  </PieChart>
              </ResponsiveContainer>
           </div>
      </Card>
    </div>

    {/* New Row: Skills & Tools */}
    <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
       <Card title="年度工具使用热度 (Top 5)">
            <div className="h-64">
              <ResponsiveContainer width="100%" height="100%">
                <BarChart data={stats.toolData.slice(0, 5)} layout="vertical">
                   <CartesianGrid strokeDasharray="3 3" horizontal={false} />
                   <XAxis type="number" hide />
                   <YAxis dataKey="name" type="category" width={80} tick={{fontSize: 12}} />
                   <RechartsTooltip />
                   <Bar dataKey="count" fill="#8884d8" name="使用频次" radius={[0, 4, 4, 0]}>
                      {stats.toolData.map((entry, index) => (
                        <Cell key={`cell-${index}`} fill={TOOL_COLORS[index % TOOL_COLORS.length]} />
                      ))}
                   </Bar>
                </BarChart>
              </ResponsiveContainer>
            </div>
       </Card>

       <Card title="技能树与能力沉淀">
          <div className="flex flex-wrap gap-2 h-64 overflow-y-auto content-start">
              {stats.uniqueSkills.length > 0 ? (
                stats.uniqueSkills.map((skill, idx) => (
                  <span key={idx} className="px-3 py-1.5 bg-blue-50 text-blue-600 rounded-lg text-sm font-medium border border-blue-100 flex items-center gap-1">
                     <Zap size={12} /> {skill}
                  </span>
                ))
              ) : (
                <div className="text-gray-400 text-sm italic w-full text-center mt-10">
                  暂无技能数据，请在“项目履历”中补充
                </div>
              )}
          </div>
       </Card>
    </div>
  </div>
);

// Milestone Drawer Component
const MilestoneDrawer = ({ project, onClose, onUpdate }) => {
    if (!project) return null;

    const milestones = project.milestones || [];

    const addMilestone = () => {
        const newMs = { date: new Date().toISOString().split('T')[0], title: "新节点", type: "milestone" };
        onUpdate(project.id, 'milestones', [...milestones, newMs]);
    };

    const updateMs = (index, field, value) => {
        const newMsList = [...milestones];
        newMsList[index] = { ...newMsList[index], [field]: value };
        onUpdate(project.id, 'milestones', newMsList.sort((a,b) => new Date(a.date) - new Date(b.date)));
    };

    const deleteMs = (index) => {
        const newMsList = milestones.filter((_, i) => i !== index);
        onUpdate(project.id, 'milestones', newMsList);
    };

    return (
        <div className="fixed inset-y-0 right-0 w-96 bg-white shadow-2xl border-l border-gray-200 p-6 z-50 overflow-y-auto animate-in slide-in-from-right duration-300">
            <div className="flex justify-between items-center mb-6">
                <div>
                    <h3 className="text-xl font-bold text-gray-800">项目大事记</h3>
                    <p className="text-sm text-gray-500 truncate w-60">{project.name}</p>
                </div>
                <button onClick={onClose} className="p-2 hover:bg-gray-100 rounded-full transition"><X size={20}/></button>
            </div>

            {/* Timeline Visual */}
            <div className="mb-8 relative pl-4 border-l-2 border-blue-100 space-y-6">
                {milestones.length === 0 && <div className="text-sm text-gray-400 italic pl-4">暂无记录，点击下方添加</div>}
                {milestones.map((ms, idx) => (
                    <div key={idx} className="relative pl-6">
                        <div className={`absolute -left-[21px] top-1 w-3 h-3 rounded-full border-2 border-white shadow-sm 
                            ${ms.type === 'start' ? 'bg-green-500' : ms.type === 'end' ? 'bg-blue-600' : 'bg-gray-400'}`}>
                        </div>
                        <div className="text-xs text-gray-400 font-mono mb-1">{ms.date}</div>
                        <div className="font-medium text-gray-800">{ms.title}</div>
                    </div>
                ))}
            </div>

            {/* Edit List */}
            <div className="space-y-4">
                <div className="flex justify-between items-center border-b border-gray-100 pb-2">
                    <h4 className="font-semibold text-gray-700 text-sm">节点管理</h4>
                    <button onClick={addMilestone} className="text-xs bg-blue-50 text-blue-600 px-2 py-1 rounded hover:bg-blue-100 flex items-center gap-1">
                        <Plus size={12}/> 添加节点
                    </button>
                </div>
                {milestones.map((ms, idx) => (
                    <div key={idx} className="flex gap-2 items-start group">
                        <input 
                            type="date" 
                            value={ms.date}
                            onChange={(e) => updateMs(idx, 'date', e.target.value)}
                            className="w-24 text-xs border border-gray-200 rounded p-1 text-gray-600"
                        />
                        <input 
                            type="text" 
                            value={ms.title}
                            onChange={(e) => updateMs(idx, 'title', e.target.value)}
                            className="flex-1 text-xs border border-gray-200 rounded p-1 text-gray-800"
                            placeholder="事件名称"
                        />
                         <button onClick={() => deleteMs(idx)} className="p-1 text-gray-300 hover:text-red-500 opacity-0 group-hover:opacity-100 transition">
                            <Trash2 size={14}/>
                        </button>
                    </div>
                ))}
            </div>
        </div>
    );
};

const DataEntryView = ({ projects, currentYear, handleAddProject, updateProject, deleteProject, globalTools }) => {
    const [editingMilestoneId, setEditingMilestoneId] = useState(null);

    const handleToolSelect = (projectId, currentTools, newTool) => {
        const toolsArr = currentTools ? currentTools.split(/[,，]/).map(t => t.trim()) : [];
        if (!toolsArr.includes(newTool)) {
            const newVal = [...toolsArr, newTool].join(', ');
            updateProject(projectId, 'tools', newVal);
        }
    };

    const currentEditingProject = projects.find(p => p.id === editingMilestoneId);

    return (
        <div className="space-y-6 relative">
            <div className="flex justify-between items-center bg-white p-4 rounded-xl border border-gray-100">
                <div>
                    <h2 className="text-xl font-bold text-gray-800">项目履历表</h2>
                    <p className="text-sm text-gray-500">在此录入{currentYear}年的所有项目，看板将自动计算。</p>
                </div>
                <button onClick={handleAddProject} className="bg-blue-600 text-white px-4 py-2 rounded-lg flex items-center gap-2 hover:bg-blue-700 transition">
                    <Plus size={16} /> 新增项目
                </button>
            </div>

            <div className="bg-white rounded-xl shadow-sm border border-gray-100 overflow-hidden">
                <div className="overflow-x-auto min-h-[400px]">
                    <table className="w-full text-sm text-left">
                        <thead className="bg-gray-50 text-gray-600 font-medium border-b border-gray-200">
                            <tr>
                                <th className="p-4 w-12">大事记</th>
                                <th className="p-4 min-w-[150px]">项目名称</th>
                                <th className="p-4 w-24">甲方/客户</th>
                                <th className="p-4 w-24">行业</th>
                                <th className="p-4 min-w-[150px]">
                                    <div className="flex items-center gap-1">核心技能/沉淀 <Zap size={12} className="text-blue-500"/></div>
                                </th>
                                <th className="p-4 min-w-[200px]">
                                    <div className="flex items-center gap-1">主要工具 <Wrench size={12} className="text-gray-500"/></div>
                                </th>
                                <th className="p-4 w-24 text-right">预估/实际(天)</th>
                                <th className="p-4 w-12">操作</th>
                            </tr>
                        </thead>
                        <tbody className="divide-y divide-gray-100">
                            {projects.map(p => (
                                <tr key={p.id} className="hover:bg-gray-50 group">
                                    <td className="p-3 align-top text-center">
                                        <button 
                                            onClick={() => setEditingMilestoneId(p.id)}
                                            className="text-gray-400 hover:text-blue-600 p-2 rounded hover:bg-blue-50 transition relative"
                                            title="编辑项目历程"
                                        >
                                            <History size={18} />
                                            {p.milestones?.length > 0 && <span className="absolute top-1 right-1 w-2 h-2 bg-blue-500 rounded-full"></span>}
                                        </button>
                                    </td>
                                    <td className="p-3 align-top">
                                        <input 
                                            type="text" 
                                            value={p.name} 
                                            onChange={(e) => updateProject(p.id, 'name', e.target.value)}
                                            className="w-full bg-transparent border-none focus:ring-0 font-medium text-gray-800 mb-1"
                                            placeholder="项目名称"
                                        />
                                        <div className="flex gap-2">
                                            <input 
                                                type="date" 
                                                value={p.endDate} 
                                                onChange={(e) => updateProject(p.id, 'endDate', e.target.value)}
                                                className="text-xs text-gray-400 bg-transparent border-none p-0 focus:ring-0"
                                            />
                                        </div>
                                    </td>
                                    <td className="p-3 align-top">
                                        <input 
                                            type="text" 
                                            value={p.client} 
                                            onChange={(e) => updateProject(p.id, 'client', e.target.value)}
                                            className="w-full bg-transparent border-none focus:ring-0 text-gray-600"
                                            placeholder="客户名"
                                        />
                                    </td>
                                    <td className="p-3 align-top">
                                        <input 
                                            type="text" 
                                            value={p.industry} 
                                            onChange={(e) => updateProject(p.id, 'industry', e.target.value)}
                                            className="w-full bg-transparent border-none focus:ring-0 text-gray-600"
                                            placeholder="行业"
                                        />
                                    </td>
                                    <td className="p-3 align-top">
                                        <textarea 
                                            value={p.skills || ''}
                                            onChange={(e) => updateProject(p.id, 'skills', e.target.value)}
                                            className="w-full bg-blue-50/50 border-none rounded focus:ring-1 focus:ring-blue-200 text-gray-600 text-xs resize-none h-16 p-2"
                                            placeholder="逗号分隔"
                                        />
                                    </td>
                                    <td className="p-3 align-top">
                                        <textarea 
                                            value={p.tools || ''}
                                            onChange={(e) => updateProject(p.id, 'tools', e.target.value)}
                                            className="w-full bg-gray-50 border-none rounded focus:ring-1 focus:ring-gray-200 text-gray-600 text-xs resize-none h-16 p-2 mb-2"
                                            placeholder="输入或点击下方标签"
                                        />
                                        {/* Tool Suggestions */}
                                        <div className="flex flex-wrap gap-1 max-w-[200px]">
                                            {globalTools.slice(0, 6).map(tool => (
                                                <button 
                                                    key={tool}
                                                    onClick={() => handleToolSelect(p.id, p.tools, tool)}
                                                    className="px-1.5 py-0.5 bg-gray-100 hover:bg-blue-100 text-gray-500 hover:text-blue-600 text-[10px] rounded border border-gray-200 transition"
                                                >
                                                    {tool}
                                                </button>
                                            ))}
                                        </div>
                                    </td>
                                    <td className="p-3 align-top text-right">
                                        <div className="flex flex-col gap-1 items-end">
                                            <div className="flex items-center gap-1 text-xs text-gray-400">
                                                <span>预</span>
                                                <input 
                                                    type="number" 
                                                    value={p.plannedDays} 
                                                    onChange={(e) => updateProject(p.id, 'plannedDays', Number(e.target.value))}
                                                    className="w-10 bg-transparent border-b border-gray-200 text-right focus:ring-0 p-0 text-sm"
                                                />
                                            </div>
                                            <div className="flex items-center gap-1 text-sm font-medium">
                                                <span className="text-xs text-gray-400">实</span>
                                                <input 
                                                    type="number" 
                                                    value={p.actualDays} 
                                                    onChange={(e) => updateProject(p.id, 'actualDays', Number(e.target.value))}
                                                    className={`w-10 bg-transparent border-b border-gray-200 text-right focus:ring-0 p-0 ${p.actualDays > p.plannedDays ? 'text-red-500' : 'text-gray-800'}`}
                                                />
                                            </div>
                                            <span className={`text-[10px] px-1.5 py-0.5 rounded 
                                                ${p.plannedDays / p.actualDays < 0.9 ? 'bg-red-100 text-red-700' : 
                                                  p.plannedDays / p.actualDays > 1.1 ? 'bg-green-100 text-green-700' : 
                                                  'bg-gray-100 text-gray-500'}`}>
                                                Eff: {p.actualDays > 0 ? Math.round((p.plannedDays / p.actualDays) * 100) : 0}%
                                            </span>
                                        </div>
                                    </td>
                                    <td className="p-3 align-top text-center pt-4">
                                        <button onClick={() => deleteProject(p.id)} className="text-gray-300 hover:text-red-500 transition">
                                            <Trash2 size={16} />
                                        </button>
                                    </td>
                                </tr>
                            ))}
                        </tbody>
                    </table>
                    {projects.length === 0 && <div className="p-8 text-center text-gray-400">点击右上方“新增项目”开始记录</div>}
                </div>
            </div>
            
            {/* Milestone Drawer Overlay */}
            {editingMilestoneId && (
                <>
                    <div 
                        className="fixed inset-0 bg-black/20 z-40 animate-in fade-in"
                        onClick={() => setEditingMilestoneId(null)}
                    ></div>
                    <MilestoneDrawer 
                        project={currentEditingProject} 
                        onClose={() => setEditingMilestoneId(null)} 
                        onUpdate={updateProject}
                    />
                </>
            )}
        </div>
    );
};

const RetrospectiveView = ({ bugs, setBugs, highlights, setHighlights }) => {
    const [searchQuery, setSearchQuery] = useState("");

    const filteredBugs = bugs.filter(b => 
        (b.issue + b.project + b.cause + b.solution).toLowerCase().includes(searchQuery.toLowerCase())
    );
    
    const filteredHighlights = highlights.filter(h => 
        (h.title + h.content + h.category).toLowerCase().includes(searchQuery.toLowerCase())
    );

    return (
        <div className="space-y-6">
            {/* Search Bar */}
            <div className="relative">
                <Search className="absolute left-3 top-1/2 -translate-y-1/2 text-gray-400" size={20} />
                <input 
                    type="text" 
                    placeholder="搜索 Bug、亮点、项目名..." 
                    className="w-full pl-10 pr-4 py-3 rounded-xl border border-gray-200 focus:border-blue-500 focus:ring-2 focus:ring-blue-100 outline-none transition"
                    value={searchQuery}
                    onChange={(e) => setSearchQuery(e.target.value)}
                />
            </div>

            <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
                {/* Workflow Issues & Bugs */}
                <div className="space-y-4">
                    <div className="flex justify-between items-center">
                        <h3 className="text-lg font-bold text-gray-800 flex items-center gap-2">
                            <Bug size={20} className="text-red-500" />
                            问题与改进 (Start/Stop)
                        </h3>
                        <button 
                            onClick={() => setBugs([...bugs, { id: Date.now(), project: "", issue: "新问题...", cause: "", solution: "" }])}
                            className="text-sm bg-red-50 text-red-600 px-3 py-1 rounded hover:bg-red-100 font-medium"
                        >
                            + 添加记录
                        </button>
                    </div>
                    <div className="space-y-3">
                        {filteredBugs.length === 0 && <div className="text-gray-400 text-center py-4 text-sm">无匹配记录</div>}
                        {filteredBugs.map(bug => (
                            <div key={bug.id} className="bg-white p-4 rounded-xl border border-red-100 shadow-sm group">
                                <div className="flex justify-between mb-2">
                                    <input 
                                        className="font-medium text-gray-800 bg-transparent w-2/3 border-b border-dashed border-gray-300 focus:border-red-500 outline-none pb-1"
                                        value={bug.issue}
                                        onChange={(e) => setBugs(bugs.map(b => b.id === bug.id ? {...b, issue: e.target.value} : b))}
                                        placeholder="输入问题描述..."
                                    />
                                    <button onClick={() => setBugs(bugs.filter(b => b.id !== bug.id))} className="text-gray-300 hover:text-red-500 opacity-0 group-hover:opacity-100 transition"><Trash2 size={14} /></button>
                                </div>
                                <div className="grid grid-cols-1 gap-2 text-sm">
                                    <div className="flex gap-2">
                                        <span className="text-gray-400 w-12 shrink-0">项目:</span>
                                        <input 
                                            className="bg-gray-50 px-2 rounded w-full text-gray-600"
                                            value={bug.project}
                                            onChange={(e) => setBugs(bugs.map(b => b.id === bug.id ? {...b, project: e.target.value} : b))}
                                            placeholder="关联项目"
                                        />
                                    </div>
                                    <div className="flex gap-2">
                                        <span className="text-gray-400 w-12 shrink-0">归因:</span>
                                        <input 
                                            className="bg-gray-50 px-2 rounded w-full text-gray-600"
                                            value={bug.cause}
                                            onChange={(e) => setBugs(bugs.map(b => b.id === bug.id ? {...b, cause: e.target.value} : b))}
                                            placeholder="根本原因..."
                                        />
                                    </div>
                                    <div className="flex gap-2 mt-2 pt-2 border-t border-gray-50">
                                        <span className="text-green-600 font-medium w-12 shrink-0">对策:</span>
                                        <textarea 
                                            className="bg-green-50 px-2 rounded w-full text-gray-700 h-16 resize-none"
                                            value={bug.solution}
                                            onChange={(e) => setBugs(bugs.map(b => b.id === bug.id ? {...b, solution: e.target.value} : b))}
                                            placeholder="明年如何避免..."
                                        />
                                    </div>
                                </div>
                            </div>
                        ))}
                    </div>
                </div>

                {/* Highlights & Methodology */}
                <div className="space-y-4">
                    <div className="flex justify-between items-center">
                        <h3 className="text-lg font-bold text-gray-800 flex items-center gap-2">
                            <Lightbulb size={20} className="text-yellow-500" />
                            亮点与SOP (Continue)
                        </h3>
                         <button 
                            onClick={() => setHighlights([...highlights, { id: Date.now(), category: "方法论", title: "新亮点...", content: "" }])}
                            className="text-sm bg-yellow-50 text-yellow-600 px-3 py-1 rounded hover:bg-yellow-100 font-medium"
                        >
                            + 添加亮点
                        </button>
                    </div>
                    <div className="space-y-3">
                        {filteredHighlights.length === 0 && <div className="text-gray-400 text-center py-4 text-sm">无匹配记录</div>}
                        {filteredHighlights.map(item => (
                            <div key={item.id} className="bg-white p-4 rounded-xl border border-yellow-100 shadow-sm group">
                                 <div className="flex justify-between mb-2">
                                    <div className="flex gap-2 items-center w-full">
                                        <select 
                                            className="text-xs bg-yellow-100 text-yellow-800 rounded px-1 py-0.5 border-none outline-none"
                                            value={item.category}
                                            onChange={(e) => setHighlights(highlights.map(h => h.id === item.id ? {...h, category: e.target.value} : h))}
                                        >
                                            <option>方法论</option>
                                            <option>工具</option>
                                            <option>沟通</option>
                                            <option>资源</option>
                                        </select>
                                        <input 
                                            className="font-medium text-gray-800 bg-transparent w-full outline-none"
                                            value={item.title}
                                            onChange={(e) => setHighlights(highlights.map(h => h.id === item.id ? {...h, title: e.target.value} : h))}
                                            placeholder="亮点标题..."
                                        />
                                    </div>
                                    <button onClick={() => setHighlights(highlights.filter(h => h.id !== item.id))} className="text-gray-300 hover:text-red-500 opacity-0 group-hover:opacity-100 transition"><Trash2 size={14} /></button>
                                </div>
                                <textarea 
                                    className="w-full text-sm text-gray-600 bg-gray-50 rounded p-2 resize-none h-20 outline-none focus:bg-white focus:ring-1 focus:ring-yellow-200"
                                    value={item.content}
                                    onChange={(e) => setHighlights(highlights.map(h => h.id === item.id ? {...h, content: e.target.value} : h))}
                                    placeholder="描述这个亮点或方法..."
                                />
                            </div>
                        ))}
                    </div>
                </div>
            </div>
        </div>
    );
};

export default function ProjectReviewDashboard() {
  const [activeTab, setActiveTab] = useState('dashboard');
  const [currentYear, setCurrentYear] = useState(2025);
  
  // Data State
  const [projects, setProjects] = useState(INITIAL_PROJECTS);
  const [bugs, setBugs] = useState(INITIAL_BUGS);
  const [highlights, setHighlights] = useState(INITIAL_HIGHLIGHTS);

  // --- Derived Statistics (Auto-calculation) ---

  const stats = useMemo(() => {
    const totalProjects = projects.length;
    const totalActualDays = projects.reduce((acc, curr) => acc + Number(curr.actualDays || 0), 0);
    const totalPlannedDays = projects.reduce((acc, curr) => acc + Number(curr.plannedDays || 0), 0);
    
    // Efficiency
    const efficiencyRate = totalActualDays > 0 ? Math.round((totalPlannedDays / totalActualDays) * 100) : 0;
    
    // Industry Distribution
    const industryDist = projects.reduce((acc, curr) => {
      acc[curr.industry] = (acc[curr.industry] || 0) + 1;
      return acc;
    }, {});
    
    const industryData = Object.keys(industryDist).map((key, index) => ({
      name: key,
      value: industryDist[key]
    }));

    // Tool Frequency
    const toolCounts = {};
    projects.forEach(p => {
      if (p.tools) {
        const toolsList = p.tools.split(/[,，、]/).map(t => t.trim()).filter(Boolean);
        toolsList.forEach(t => {
          toolCounts[t] = (toolCounts[t] || 0) + 1;
        });
      }
    });
    const toolData = Object.keys(toolCounts)
      .map(key => ({ name: key, count: toolCounts[key] }))
      .sort((a, b) => b.count - a.count);

    // Unique Skills
    const skillSet = new Set();
    projects.forEach(p => {
      if (p.skills) {
        const skillsList = p.skills.split(/[,，、]/).map(s => s.trim()).filter(Boolean);
        skillsList.forEach(s => skillSet.add(s));
      }
    });
    const uniqueSkills = Array.from(skillSet);

    // Monthly completion trend
    const monthlyData = Array(12).fill(0).map((_, i) => ({ name: `${i+1}月`, count: 0, efficiency: 0 }));
    projects.forEach(p => {
        if(!p.endDate) return;
        const month = new Date(p.endDate).getMonth();
        if (month >= 0 && month < 12) {
            monthlyData[month].count += 1;
            const eff = p.actualDays > 0 ? (p.plannedDays / p.actualDays) * 100 : 0;
            monthlyData[month].efficiency = monthlyData[month].efficiency === 0 ? eff : (monthlyData[month].efficiency + eff) / 2;
        }
    });

    return { totalProjects, totalActualDays, efficiencyRate, industryData, monthlyData, toolData, uniqueSkills };
  }, [projects]);

  // --- Handlers ---

  const handleAddProject = () => {
    const newId = projects.length > 0 ? Math.max(...projects.map(p => p.id)) + 1 : 1;
    setProjects([...projects, { 
      id: newId, 
      name: "", 
      client: "", 
      industry: "", 
      startDate: "", 
      endDate: "", 
      plannedDays: 0, 
      actualDays: 0, 
      status: "进行中", 
      type: "迭代",
      skills: "",
      tools: "",
      milestones: [] // New field
    }]);
  };

  const updateProject = (id, field, value) => {
    setProjects(projects.map(p => p.id === id ? { ...p, [field]: value } : p));
  };

  const deleteProject = (id) => {
    setProjects(projects.filter(p => p.id !== id));
  };

  // --- Main Layout ---

  return (
    <div className="min-h-screen bg-slate-50 flex font-sans text-gray-900">
      {/* Sidebar */}
      <div className="w-64 bg-slate-900 text-white flex flex-col hidden md:flex shrink-0">
        <div className="p-6">
          <h1 className="text-xl font-bold tracking-tight">PM 年度复盘</h1>
          <p className="text-slate-400 text-xs mt-1">Professional Review Kit</p>
        </div>
        
        <nav className="flex-1 px-4 space-y-2">
          <button 
            onClick={() => setActiveTab('dashboard')}
            className={`w-full flex items-center gap-3 px-4 py-3 rounded-lg text-sm font-medium transition-colors ${activeTab === 'dashboard' ? 'bg-blue-600 text-white' : 'text-slate-400 hover:bg-slate-800 hover:text-white'}`}
          >
            <LayoutDashboard size={18} /> 全局看板
          </button>
          <button 
            onClick={() => setActiveTab('data')}
            className={`w-full flex items-center gap-3 px-4 py-3 rounded-lg text-sm font-medium transition-colors ${activeTab === 'data' ? 'bg-blue-600 text-white' : 'text-slate-400 hover:bg-slate-800 hover:text-white'}`}
          >
            <ListTodo size={18} /> 项目履历
          </button>
          <button 
            onClick={() => setActiveTab('retro')}
            className={`w-full flex items-center gap-3 px-4 py-3 rounded-lg text-sm font-medium transition-colors ${activeTab === 'retro' ? 'bg-blue-600 text-white' : 'text-slate-400 hover:bg-slate-800 hover:text-white'}`}
          >
            <TrendingUp size={18} /> 复盘与沉淀
          </button>
        </nav>

        <div className="p-4 border-t border-slate-800">
            <div className="flex items-center justify-between text-slate-400 text-sm mb-2">
                <span>年份切换</span>
                <Calendar size={14} />
            </div>
            <div className="flex gap-2">
                {[2023, 2024, 2025].map(year => (
                    <button 
                        key={year}
                        onClick={() => setCurrentYear(year)}
                        className={`px-3 py-1 rounded text-xs font-bold ${currentYear === year ? 'bg-white text-slate-900' : 'bg-slate-800 text-slate-400 hover:bg-slate-700'}`}
                    >
                        {year}
                    </button>
                ))}
            </div>
        </div>
      </div>

      {/* Main Content */}
      <div className="flex-1 flex flex-col h-screen overflow-hidden">
        {/* Mobile Header */}
        <div className="md:hidden bg-slate-900 text-white p-4 flex justify-between items-center">
             <h1 className="font-bold">PM 复盘 {currentYear}</h1>
             <div className="flex gap-2">
                <button onClick={() => setActiveTab('dashboard')} className="p-2"><LayoutDashboard size={20}/></button>
                <button onClick={() => setActiveTab('data')} className="p-2"><ListTodo size={20}/></button>
             </div>
        </div>

        {/* Content Area */}
        <main className="flex-1 overflow-y-auto p-4 md:p-8">
            <header className="mb-8 flex justify-between items-end">
                <div>
                    <h2 className="text-2xl font-bold text-gray-800">
                        {activeTab === 'dashboard' && '全局概览'}
                        {activeTab === 'data' && '项目数据库'}
                        {activeTab === 'retro' && '价值沉淀'}
                    </h2>
                    <p className="text-gray-500 mt-1">
                        {activeTab === 'dashboard' && `查看 ${currentYear} 年度核心指标与分布`}
                        {activeTab === 'data' && '管理所有参与项目的详细量化数据'}
                        {activeTab === 'retro' && '梳理问题，总结亮点，为明年做准备'}
                    </p>
                </div>
                <div className="hidden md:flex gap-3">
                    <button className="flex items-center gap-2 text-sm text-gray-500 hover:text-blue-600 px-3 py-2 rounded-lg hover:bg-blue-50 transition">
                        <Download size={16} /> 导出报告
                    </button>
                    <button className="flex items-center gap-2 text-sm bg-slate-900 text-white px-4 py-2 rounded-lg hover:bg-slate-800 transition">
                        <Save size={16} /> 保存快照
                    </button>
                </div>
            </header>

            {activeTab === 'dashboard' && <DashboardView stats={stats} projects={projects} />}
            {activeTab === 'data' && <DataEntryView 
                projects={projects} 
                currentYear={currentYear}
                handleAddProject={handleAddProject} 
                updateProject={updateProject} 
                deleteProject={deleteProject}
                globalTools={stats.toolData.map(t => t.name)}
            />}
            {activeTab === 'retro' && <RetrospectiveView 
                bugs={bugs} 
                setBugs={setBugs} 
                highlights={highlights} 
                setHighlights={setHighlights} 
            />}
        </main>
      </div>
    </div>
  );
}
