import React, { useEffect, useRef, useState } from 'react';
import { Send, Settings, Trash2, Download, Copy, Menu, User, Bot, Moon, Sun, Monitor } from 'lucide-react';

export default function ARIAMaterialChat() {
  // Theme mode: 'light' | 'dark' | 'auto'
  const [themeMode, setThemeMode] = useState('dark');
  
  // Drawer
  const [drawerOpen, setDrawerOpen] = useState(true);
  
  // Chat message state
  const [messages, setMessages] = useState([]);
  const [input, setInput] = useState('');
  const [loading, setLoading] = useState(false);
  
  // LangChain config
  const [systemPrompt, setSystemPrompt] = useState('You are ARIA (Advanced Reasoning Intelligence Avatar), a sophisticated AI assistant. Provide detailed, well-structured responses.');
  const [temperature, setTemperature] = useState(0.7);
  const [maxTokens, setMaxTokens] = useState(2000);
  
  // UI options
  const [chatStyle, setChatStyle] = useState('bubbles');
  const [showSettings, setShowSettings] = useState(false);
  
  // Stats
  const [stats, setStats] = useState({
    totalMessages: 0,
    totalTokens: 0,
    sessionStart: new Date().toLocaleTimeString()
  });
  
  // refs
  const endRef = useRef(null);
  const conversationHistory = useRef([]);

  useEffect(() => {
    const welcome = {
      role: 'assistant',
      content: 'Hello — this is ARIA. How can I help you today?',
      ts: Date.now(),
    };
    setMessages([welcome]);
    conversationHistory.current = [welcome];
  }, []);

  useEffect(() => {
    endRef.current?.scrollIntoView({ behavior: 'smooth' });
  }, [messages, loading]);

  // LangChain memory management
  const getConversationMemory = () => {
    const memoryWindow = conversationHistory.current.slice(-10);
    return memoryWindow.map(m => ({
      role: m.role,
      content: m.content
    }));
  };

  const handleSend = async () => {
    if (!input.trim() || loading) return;
    
    const userMsg = { role: 'user', content: input.trim(), ts: Date.now() };
    setMessages(prev => [...prev, userMsg]);
    conversationHistory.current.push(userMsg);
    setInput('');
    setLoading(true);

    setStats(prev => ({
      ...prev,
      totalMessages: prev.totalMessages + 1,
      totalTokens: prev.totalTokens + input.length
    }));

    try {
      const context = getConversationMemory();
      
      const response = await fetch('https://api.anthropic.com/v1/messages', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          model: 'claude-sonnet-4-20250514',
          max_tokens: maxTokens,
          temperature: temperature,
          system: systemPrompt,
          messages: [...context, { role: 'user', content: userMsg.content }]
        })
      });

      const data = await response.json();
      const assistantContent = data.content[0].text;
      
      const assistantMsg = {
        role: 'assistant',
        content: assistantContent,
        ts: Date.now()
      };

      conversationHistory.current.push(assistantMsg);
      setMessages(prev => [...prev, assistantMsg]);
      
      setStats(prev => ({
        ...prev,
        totalMessages: prev.totalMessages + 1,
        totalTokens: prev.totalTokens + assistantContent.length
      }));

    } catch (err) {
      const errorMsg = {
        role: 'assistant',
        content: '⚠️ Error: Unable to generate response. Please try again.',
        ts: Date.now(),
        error: true
      };
      setMessages(prev => [...prev, errorMsg]);
      conversationHistory.current.push(errorMsg);
    } finally {
      setLoading(false);
    }
  };

  const handleClear = () => {
    const welcome = {
      role: 'assistant',
      content: '🔄 Conversation reset. How can I help you?',
      ts: Date.now(),
    };
    setMessages([welcome]);
    conversationHistory.current = [welcome];
    setStats(prev => ({
      ...prev,
      totalMessages: 0,
      totalTokens: 0,
      sessionStart: new Date().toLocaleTimeString()
    }));
  };

  const handleExport = () => {
    const exportData = {
      conversation: conversationHistory.current,
      stats: stats,
      config: { systemPrompt, temperature, maxTokens },
      exportedAt: new Date().toISOString()
    };
    const blob = new Blob([JSON.stringify(exportData, null, 2)], { type: 'application/json' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `aria-chat-${Date.now()}.json`;
    a.click();
  };

  const copyMessage = (content) => {
    navigator.clipboard.writeText(content);
  };

  const handleKeyPress = (e) => {
    if (e.key === 'Enter' && !e.shiftKey) {
      e.preventDefault();
      handleSend();
    }
  };

  // Theme colors
  const isDark = themeMode === 'dark';
  const bgPrimary = isDark ? 'bg-gray-900' : 'bg-gray-50';
  const bgSecondary = isDark ? 'bg-gray-800' : 'bg-white';
  const bgTertiary = isDark ? 'bg-gray-700' : 'bg-gray-100';
  const textPrimary = isDark ? 'text-white' : 'text-gray-900';
  const textSecondary = isDark ? 'text-gray-300' : 'text-gray-600';
  const textTertiary = isDark ? 'text-gray-400' : 'text-gray-500';
  const border = isDark ? 'border-gray-700' : 'border-gray-200';

  return (
    <div className={`h-screen flex ${bgPrimary}`}>
      
      {/* Drawer/Sidebar */}
      {drawerOpen && (
        <div className={`w-80 ${bgSecondary} border-r ${border} flex flex-col`}>
          {/* Drawer Header */}
          <div className="p-4 border-b ${border}">
            <div className="flex items-center justify-between mb-4">
              <div className="flex items-center gap-3">
                <div className="w-10 h-10 bg-gradient-to-br from-blue-500 to-purple-600 rounded-lg flex items-center justify-center">
                  <Bot className="w-6 h-6 text-white" />
                </div>
                <div>
                  <h2 className={`font-bold ${textPrimary}`}>ARIA</h2>
                  <p className={`text-xs ${textTertiary}`}>Gen AI Avatar</p>
                </div>
              </div>
              <button onClick={() => setShowSettings(!showSettings)} className={`p-2 rounded-lg ${bgTertiary} ${textSecondary}`}>
                <Settings className="w-4 h-4" />
              </button>
            </div>

            {/* Theme Toggle */}
            <div className="flex gap-2">
              <button
                onClick={() => setThemeMode('light')}
                className={`flex-1 p-2 rounded-lg flex items-center justify-center gap-2 ${themeMode === 'light' ? 'bg-blue-500 text-white' : `${bgTertiary} ${textSecondary}`}`}
              >
                <Sun className="w-4 h-4" />
                <span className="text-xs">Light</span>
              </button>
              <button
                onClick={() => setThemeMode('dark')}
                className={`flex-1 p-2 rounded-lg flex items-center justify-center gap-2 ${themeMode === 'dark' ? 'bg-blue-500 text-white' : `${bgTertiary} ${textSecondary}`}`}
              >
                <Moon className="w-4 h-4" />
                <span className="text-xs">Dark</span>
              </button>
            </div>
          </div>

          {/* Stats Card */}
          <div className="p-4">
            <div className={`${bgTertiary} rounded-lg p-4`}>
              <h3 className={`font-semibold mb-3 ${textPrimary}`}>Session Stats</h3>
              <div className="space-y-2 text-sm">
                <div className={`flex justify-between ${textSecondary}`}>
                  <span>Messages</span>
                  <span className="text-blue-500 font-semibold">{stats.totalMessages}</span>
                </div>
                <div className={`flex justify-between ${textSecondary}`}>
                  <span>Tokens</span>
                  <span className="text-purple-500 font-semibold">~{stats.totalTokens}</span>
                </div>
                <div className={`flex justify-between ${textSecondary}`}>
                  <span>Memory</span>
                  <span className="text-green-500 font-semibold">{conversationHistory.current.length}/10</span>
                </div>
                <div className={`flex justify-between ${textSecondary}`}>
                  <span>Started</span>
                  <span className="text-xs">{stats.sessionStart}</span>
                </div>
              </div>
            </div>
          </div>

          {/* Settings Panel */}
          {showSettings && (
            <div className="px-4 pb-4 flex-1 overflow-y-auto">
              <div className={`${bgTertiary} rounded-lg p-4`}>
                <h3 className={`font-semibold mb-3 ${textPrimary}`}>LangChain Config</h3>
                
                <div className="space-y-4">
                  <div>
                    <label className={`text-sm ${textSecondary} block mb-2`}>
                      Temperature: {temperature.toFixed(1)}
                    </label>
                    <input
                      type="range"
                      min="0"
                      max="1"
                      step="0.1"
                      value={temperature}
                      onChange={(e) => setTemperature(parseFloat(e.target.value))}
                      className="w-full"
                    />
                  </div>

                  <div>
                    <label className={`text-sm ${textSecondary} block mb-2`}>
                      Max Tokens: {maxTokens}
                    </label>
                    <input
                      type="range"
                      min="500"
                      max="4000"
                      step="500"
                      value={maxTokens}
                      onChange={(e) => setMaxTokens(parseInt(e.target.value))}
                      className="w-full"
                    />
                  </div>

                  <div>
                    <label className={`text-sm ${textSecondary} block mb-2`}>Chat Style</label>
                    <select
                      value={chatStyle}
                      onChange={(e) => setChatStyle(e.target.value)}
                      className={`w-full ${bgSecondary} ${textPrimary} border ${border} rounded-lg p-2 text-sm`}
                    >
                      <option value="bubbles">Bubbles</option>
                      <option value="cards">Cards</option>
                      <option value="minimal">Minimal</option>
                    </select>
                  </div>

                  <div>
                    <label className={`text-sm ${textSecondary} block mb-2`}>System Prompt</label>
                    <textarea
                      value={systemPrompt}
                      onChange={(e) => setSystemPrompt(e.target.value)}
                      className={`w-full ${bgSecondary} ${textPrimary} border ${border} rounded-lg p-2 text-xs h-24 resize-none`}
                    />
                  </div>
                </div>
              </div>
            </div>
          )}

          {/* Action Buttons */}
          <div className="p-4 space-y-2 border-t ${border}">
            <button
              onClick={handleExport}
              className="w-full bg-blue-600 hover:bg-blue-700 text-white py-2 px-4 rounded-lg flex items-center justify-center gap-2 transition-colors"
            >
              <Download className="w-4 h-4" />
              Export Chat
            </button>
            <button
              onClick={handleClear}
              className={`w-full ${isDark ? 'bg-red-600/20 hover:bg-red-600/30 text-red-400 border-red-600/30' : 'bg-red-50 hover:bg-red-100 text-red-600 border-red-200'} py-2 px-4 rounded-lg flex items-center justify-center gap-2 transition-colors border`}
            >
              <Trash2 className="w-4 h-4" />
              Clear Chat
            </button>
          </div>
        </div>
      )}

      {/* Main Chat Area */}
      <div className="flex-1 flex flex-col">
        
        {/* AppBar */}
        <div className={`${bgSecondary} border-b ${border} p-4`}>
          <div className="flex items-center justify-between">
            <div className="flex items-center gap-3">
              <button
                onClick={() => setDrawerOpen(!drawerOpen)}
                className={`p-2 rounded-lg ${bgTertiary} ${textSecondary}`}
              >
                <Menu className="w-5 h-5" />
              </button>
              <div>
                <h1 className={`text-lg font-semibold ${textPrimary}`}>Advanced Gen AI Chat</h1>
                <p className={`text-xs ${textTertiary}`}>Powered by LangChain + Claude LLM</p>
              </div>
            </div>
            <div className={`px-3 py-1 rounded-full text-xs ${bgTertiary} ${textSecondary} flex items-center gap-2`}>
              <div className={`w-2 h-2 rounded-full ${loading ? 'bg-yellow-400 animate-pulse' : 'bg-green-400'}`}></div>
              {loading ? 'Processing' : 'Ready'}
            </div>
          </div>
        </div>

        {/* Messages */}
        <div className="flex-1 overflow-y-auto p-6 space-y-4">
          {messages.map((msg, idx) => {
            if (chatStyle === 'bubbles') {
              return (
                <div key={idx} className={`flex ${msg.role === 'user' ? 'justify-end' : 'justify-start'}`}>
                  <div className={`max-w-2xl flex items-start gap-3 ${msg.role === 'user' ? 'flex-row-reverse' : ''}`}>
                    <div className={`w-8 h-8 rounded-full flex items-center justify-center flex-shrink-0 ${
                      msg.role === 'user' ? 'bg-blue-600' : 'bg-gradient-to-br from-purple-600 to-blue-600'
                    }`}>
                      {msg.role === 'user' ? <User className="w-4 h-4 text-white" /> : <Bot className="w-4 h-4 text-white" />}
                    </div>
                    <div>
                      <div className={`px-4 py-3 rounded-2xl ${
                        msg.role === 'user'
                          ? 'bg-blue-600 text-white'
                          : isDark ? 'bg-gray-800 text-gray-100' : 'bg-gray-200 text-gray-900'
                      }`}>
                        <p className="text-sm whitespace-pre-wrap">{msg.content}</p>
                      </div>
                      <div className={`flex items-center gap-2 mt-1 text-xs ${textTertiary}`}>
                        <span>{new Date(msg.ts).toLocaleTimeString()}</span>
                        {msg.role === 'assistant' && (
                          <button
                            onClick={() => copyMessage(msg.content)}
                            className="hover:text-blue-500"
                          >
                            <Copy className="w-3 h-3" />
                          </button>
                        )}
                      </div>
                    </div>
                  </div>
                </div>
              );
            } else if (chatStyle === 'cards') {
              return (
                <div key={idx} className={`${bgSecondary} border ${border} rounded-lg p-4`}>
                  <div className="flex items-start gap-3">
                    <div className={`w-10 h-10 rounded-lg flex items-center justify-center ${
                      msg.role === 'user' ? 'bg-blue-600' : 'bg-gradient-to-br from-purple-600 to-blue-600'
                    }`}>
                      {msg.role === 'user' ? <User className="w-5 h-5 text-white" /> : <Bot className="w-5 h-5 text-white" />}
                    </div>
                    <div className="flex-1">
                      <div className="flex items-center justify-between mb-2">
                        <span className={`text-sm font-semibold ${textPrimary}`}>
                          {msg.role === 'user' ? 'You' : 'ARIA'}
                        </span>
                        <span className={`text-xs ${textTertiary}`}>
                          {new Date(msg.ts).toLocaleTimeString()}
                        </span>
                      </div>
                      <p className={`text-sm ${textSecondary} whitespace-pre-wrap`}>{msg.content}</p>
                      {msg.role === 'assistant' && (
                        <button
                          onClick={() => copyMessage(msg.content)}
                          className={`mt-2 text-xs ${textTertiary} hover:text-blue-500 flex items-center gap-1`}
                        >
                          <Copy className="w-3 h-3" />
                          Copy
                        </button>
                      )}
                    </div>
                  </div>
                </div>
              );
            } else {
              return (
                <div key={idx} className="space-y-1">
                  <div className={`text-xs font-semibold ${textSecondary}`}>
                    {msg.role === 'user' ? 'You' : 'ARIA'} • {new Date(msg.ts).toLocaleTimeString()}
                  </div>
                  <p className={`text-sm ${textPrimary} whitespace-pre-wrap`}>{msg.content}</p>
                </div>
              );
            }
          })}
          
          {loading && (
            <div className="flex justify-start">
              <div className={`${bgTertiary} px-4 py-3 rounded-2xl`}>
                <div className="flex items-center gap-3">
                  <div className="flex gap-1">
                    <div className="w-2 h-2 bg-blue-400 rounded-full animate-bounce"></div>
                    <div className="w-2 h-2 bg-purple-400 rounded-full animate-bounce" style={{animationDelay: '0.1s'}}></div>
                    <div className="w-2 h-2 bg-pink-400 rounded-full animate-bounce" style={{animationDelay: '0.2s'}}></div>
                  </div>
                  <span className={`text-sm ${textSecondary}`}>Processing...</span>
                </div>
              </div>
            </div>
          )}
          <div ref={endRef} />
        </div>

        {/* Input Area */}
        <div className={`${bgSecondary} border-t ${border} p-4`}>
          <div className="max-w-4xl mx-auto">
            <div className="flex gap-3 items-end">
              <textarea
                value={input}
                onChange={(e) => setInput(e.target.value)}
                onKeyPress={handleKeyPress}
                placeholder="Message ARIA... (Shift+Enter for new line)"
                rows="2"
                disabled={loading}
                className={`flex-1 ${bgTertiary} ${textPrimary} rounded-xl px-4 py-3 focus:outline-none focus:ring-2 focus:ring-blue-500 resize-none border ${border}`}
              />
              <button
                onClick={handleSend}
                disabled={loading || !input.trim()}
                className="bg-gradient-to-r from-blue-600 to-purple-600 text-white p-3 rounded-xl hover:shadow-lg hover:scale-105 transition-all disabled:opacity-50 disabled:cursor-not-allowed disabled:scale-100"
              >
                <Send className="w-5 h-5" />
              </button>
            </div>
            <div className={`mt-2 text-xs ${textTertiary} flex items-center justify-between`}>
              <span>💡 Context-aware with LangChain memory</span>
              <span>{input.length} characters</span>
            </div>
          </div>
        </div>
      </div>
    </div>
  );
}
