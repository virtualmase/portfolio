import React, { useState, useEffect, useCallback } from 'react';
// Import the new getPrompt function from the openai.js file
import { getPrompt } from './openai'; // Assuming openai.js is in the same directory as App.jsx

// Supabase Integration (Optional):
// Uncomment the following lines if you set up Supabase and want to integrate data storage.
// import supabase from './supabase'; // Assuming supabase.js is in the same directory as App.jsx

// Main App Component for the Magnetic Lighthouse with a Soul Prototype
export default function App() {
  // State for Signal Map
  const [beacons, setBeacons] = useState([
    { id: 'b1', name: 'Sovereignty', color: 'text-teal-400' },
    { id: 'b2', name: 'Decentralized Collaboration', color: 'text-blue-400' },
    { id: 'b3', name: 'Planetary Health', color: 'text-green-400' },
  ]);
  const [intentions, setIntentions] = useState([
    { id: 'i1', name: 'Web3 Grant Application', beaconId: 'b2', pull: 'High' },
    { id: 'i2', name: 'Community Governance Proposal', beaconId: 'b2', pull: 'Medium' },
    { id: 'i3', name: 'Sustainable Tech Research', beaconId: 'b3', pull: 'High' },
    { id: 'i4', name: 'Personal Growth Project', beaconId: 'b1', pull: 'Medium' },
  ]);
  const [newBeaconName, setNewBeaconName] = useState('');
  const [newIntentionName, setNewIntentionName] = useState('');
  const [selectedBeaconForIntention, setSelectedBeaconForIntention] = useState(beacons[0]?.id || '');

  // State for Soul Dialogue Console
  const [journalEntry, setJournalEntry] = useState('');
  const [aiResponse, setAiResponse] = useState(''); // Stores the latest AI response
  const [isLoadingAi, setIsLoadingAi] = useState(false);
  const [dialogueHistory, setDialogueHistory] = useState([]); // Stores user input and AI responses for the conversation flow

  // State for Opportunity Field - Initial opportunities without computed relevance
  const [rawOpportunities, setRawOpportunities] = useState([
    { id: 'o1', title: 'Decentralized Identity Grant', source: 'Gitcoin', tags: ['Web3', 'Grant', 'Sovereignty', 'Decentralized Collaboration'] },
    { id: 'o2', title: 'Climate Data Initiative Fellowship', source: 'Notion Feed', tags: ['Health', 'Fellowship', 'Planetary Health'] },
    { id: 'o3', title: 'AI Ethics Research Role', source: 'X', tags: ['AI', 'Research', 'Sovereignty'] },
    { id: 'o4', title: 'Local Community Event', source: 'Custom DB', tags: ['Community', 'Decentralized Collaboration'] },
    { id: 'o5', title: 'Open-Source Protocol Development', source: 'Gitcoin', tags: ['Web3', 'Development', 'Sovereignty', 'Decentralized Collaboration'] },
    { id: 'o6', title: 'Ecological Restoration Project', source: 'Notion Feed', tags: ['Environment', 'Planetary Health'] },
    { id: 'o7', title: 'Blockchain for Supply Chain Transparency', source: 'Lens', tags: ['Web3', 'Supply Chain', 'Sovereignty'] },
    { id: 'o8', title: 'Community-led Renewable Energy Fund', source: 'Gitcoin', tags: ['Energy', 'Community', 'Planetary Health', 'Decentralized Collaboration'] },
  ]);
  const [opportunities, setOpportunities] = useState([]); // This will store opportunities with computed relevance
  const [filterTag, setFilterTag] = useState('All');

  // State for Soulforge
  const [projectBriefInput, setProjectBriefInput] = useState('');
  const [lighthouseSeeds, setLighthouseSeeds] = useState([]); // Stores generated project sparks

  // Initialize selectedBeaconForIntention when beacons are loaded
  useEffect(() => {
    if (beacons.length > 0 && !selectedBeaconForIntention) {
      setSelectedBeaconForIntention(beacons[0].id);
    }
  }, [beacons, selectedBeaconForIntention]);

  // Semantic Scoring Engine Logic
  // This function calculates a relevance score for an opportunity based on user's defined beacons and intentions.
  const calculateOpportunityRelevance = useCallback((opportunity, currentBeacons, currentIntentions) => {
    let score = 0;
    // Combine opportunity title, source, and tags for comprehensive keyword matching
    const opportunityText = `${opportunity.title} ${opportunity.source} ${opportunity.tags.join(' ')}`.toLowerCase();

    // Iterate through current beacons to find keyword matches in the opportunity text
    currentBeacons.forEach(beacon => {
      const beaconKeywords = beacon.name.toLowerCase().split(' ').filter(word => word.length > 2); // Filter short, common words
      beaconKeywords.forEach(keyword => {
        if (opportunityText.includes(keyword)) {
          score += 0.3; // Higher weight for direct beacon keyword match
        }
      });
    });

    // Iterate through current intentions to find keyword matches
    currentIntentions.forEach(intention => {
      const intentionKeywords = intention.name.toLowerCase().split(' ').filter(word => word.length > 2);
      intentionKeywords.forEach(keyword => {
        if (opportunityText.includes(keyword)) {
          score += 0.2; // Medium weight for intention keyword match
        }
      });
    });

    // Score based on direct tag matches (simplified example, could be more complex with semantic similarity)
    opportunity.tags.forEach(tag => {
      const lowerTag = tag.toLowerCase();
      // Directly check if a tag aligns with general themes from beacons/intentions
      if (lowerTag.includes('web3') && (opportunityText.includes('decentralized') || opportunityText.includes('sovereignty'))) {
        score += 0.2;
      }
      if (lowerTag.includes('health') || lowerTag.includes('environment')) {
        if (opportunityText.includes('planetary health') || opportunityText.includes('sustainable')) {
            score += 0.2;
        }
      }
      if (lowerTag.includes('ai') && (opportunityText.includes('ethics') || opportunityText.includes('research'))) {
        score += 0.1;
      }
      if (lowerTag.includes('community') && opportunityText.includes('collaboration')) {
        score += 0.15;
      }
    });

    // Clamp score between 0 and 1, ensuring it's a fixed-point number for display
    return Math.min(1, parseFloat(score.toFixed(2)));
  }, []); // Dependencies are stable (only primitive types, no dynamic external state for the logic itself)

  // Recalculate opportunities relevance whenever beacons or intentions change
  useEffect(() => {
    // In a real application, you might fetch opportunities from a backend here
    // and then apply the relevance scoring.
    const updatedOpportunities = rawOpportunities.map(op => ({
      ...op,
      relevance: calculateOpportunityRelevance(op, beacons, intentions)
    }));
    // Sort by relevance descending to show most relevant opportunities first
    setOpportunities(updatedOpportunities.sort((a, b) => b.relevance - a.relevance));
  }, [beacons, intentions, rawOpportunities, calculateOpportunityRelevance]);


  // Function to add a new beacon to the Signal Map
  const addBeacon = () => {
    if (newBeaconName.trim()) {
      const newBeacon = {
        id: `b${beacons.length + 1}`,
        name: newBeaconName.trim(),
        // Cycle through predefined colors for visual distinction
        color: `text-${['teal', 'blue', 'green', 'purple', 'rose', 'indigo'][beacons.length % 6]}-400`,
      };
      setBeacons([...beacons, newBeacon]);
      setNewBeaconName('');
      // If it's the first beacon, auto-select it for intention creation
      if (beacons.length === 0) {
        setSelectedBeaconForIntention(newBeacon.id);
      }
      // Optional: Supabase insert for new beacon
      // const { data, error } = await supabase.from('beacons').insert([newBeacon]);
      // if (error) console.error("Error saving beacon to Supabase:", error);
    }
  };

  // Function to add a new intention to the Signal Map
  const addIntention = () => {
    if (newIntentionName.trim() && selectedBeaconForIntention) {
      const newIntention = {
        id: `i${intentions.length + 1}`,
        name: newIntentionName.trim(),
        beaconId: selectedBeaconForIntention,
        pull: 'Medium', // Default pull, could be user-selectable
      };
      setIntentions([...intentions, newIntention]);
      setNewIntentionName('');
      // Optional: Supabase insert for new intention
      // const { data, error } = await supabase.from('intentions').insert([newIntention]);
      // if (error) console.error("Error saving intention to Supabase:", error);
    }
  };

  // Handle sending journal entry to AI (GPT-4 Powered Journaling)
  const sendToSoulDialogue = async (e) => {
    e.preventDefault(); // Prevent default form submission behavior
    if (journalEntry.trim()) {
      setIsLoadingAi(true);
      const userEntryText = journalEntry.trim();
      setDialogueHistory(prev => [...prev, { type: 'user', text: userEntryText }]); // Add user's entry to history
      
      // Optional: Supabase insert for user journal entry
      // const { data: journalData, error: journalError } = await supabase
      //   .from('journals')
      //   .insert([{ user_input: userEntryText }]);
      // if (journalError) console.error("Error saving journal entry to Supabase:", journalError);

      try {
        // Call the getPrompt function from openai.js for AI insight
        const aiInsight = await getPrompt(userEntryText);
        setAiResponse(aiInsight); // Update the latest AI response state
        setDialogueHistory(prev => [...prev, { type: 'ai', text: aiInsight }]); // Add AI response to history

        // Optional: Supabase insert for AI response
        // const { data: aiResponseData, error: aiResponseError } = await supabase
        //   .from('ai_responses') // Assuming a separate table for AI responses
        //   .insert([{ journal_entry_id: journalData?.id, generated_prompt: aiInsight }]); // Link to journal entry if applicable
        // if (aiResponseError) console.error("Error saving AI response to Supabase:", aiResponseError);

      } catch (error) {
        console.error("Error sending to Soul Dialogue:", error);
        const errorMessage = "An error occurred with the AI. Please try again later.";
        setDialogueHistory(prev => [...prev, { type: 'ai', text: errorMessage }]);
      } finally {
        setIsLoadingAi(false);
        setJournalEntry(''); // Clear journal entry after sending
      }
    }
  };

  // Function to create a "Soul Seed" in the Soulforge
  const createSoulSeed = () => {
    if (projectBriefInput.trim()) {
      setLighthouseSeeds([...lighthouseSeeds, projectBriefInput.trim()]);
      setProjectBriefInput('');
      // Optional: Supabase insert for Soul Seed
      // const { data, error } = await supabase.from('soul_seeds').insert([{ seed_text: projectBriefInput.trim() }]);
      // if (error) console.error("Error saving Soul Seed to Supabase:", error);
    }
  };

  // Filter opportunities based on tag and sort by relevance
  const filteredOpportunities = opportunities
    .filter(op => filterTag === 'All' || op.tags.includes(filterTag))
    .sort((a, b) => b.relevance - a.relevance); // Ensure sorting is always by relevance


  // Utility function for text styling based on relevance/pull
  const getRelevanceStyle = (relevance) => {
    if (relevance >= 0.8) return 'text-purple-400 neon-text'; // High relevance glows more
    if (relevance >= 0.6) return 'text-blue-400';
    if (relevance >= 0.4) return 'text-yellow-400';
    return 'text-gray-400';
  };

  const getPullStyle = (pull) => {
    switch (pull) {
      case 'High': return 'text-green-400';
      case 'Medium': return 'text-yellow-400';
      case 'Low': return 'text-red-400';
      default: return 'text-gray-400';
    }
  };

  return (
    <div className="min-h-screen bg-gradient-to-br from-gray-950 to-gray-800 text-gray-100 font-sans p-4 sm:p-8 relative overflow-hidden">
      {/* Background radial pulses / aurora-like flows */}
      <div className="absolute inset-0 z-0 opacity-10">
          <div className="absolute w-80 h-80 rounded-full bg-blue-500 blur-3xl opacity-30 top-1/4 left-1/4 animate-pulse"></div>
          <div className="absolute w-96 h-96 rounded-full bg-teal-500 blur-3xl opacity-30 bottom-1/4 right-1/4 animate-pulse animation-delay-2000"></div>
          <div className="absolute w-72 h-72 rounded-full bg-indigo-500 blur-3xl opacity-20 top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 animate-pulse animation-delay-4000"></div>
      </div>

      <div className="relative z-10 max-w-7xl mx-auto space-y-12">
        <header className="text-center py-8">
          <h1 className="text-5xl sm:text-6xl font-extrabold mb-4 neon-text">
            Powering Foundational Intelligence Architectures
          </h1>
          <p className="text-xl sm:text-2xl text-gray-300">
            A Coreweaver Labs Design: Magnetic Lighthouse with a Soul
          </p>
          <p className="text-lg text-gray-400 mt-2">
            Maintain direction, coherence, and inner clarity in complex, shifting environments.
          </p>
        </header>

        {/* Vision Compass Section */}
        <section id="signal-map" className="glass-panel p-6 sm:p-10 rounded-xl shadow-lg border border-blue-700/30 flex flex-col lg:flex-row gap-8">
            <div className="lg:w-1/2">
                <h2 className="text-3xl font-bold mb-6 neon-text text-center">🧭 Signal Map: Your Vision Compass</h2>
                <p className="text-gray-300 mb-6">
                    Define your core beacons (mission, purpose, values) and orbiting intentions (projects, collaborators) to visualize your alignment.
                </p>

                {/* Beacons and Intentions Display */}
                <div className="grid grid-cols-1 md:grid-cols-2 gap-6 mb-8">
                    <div>
                        <h3 className="text-2xl font-semibold text-teal-300 mb-3">Core Beacons</h3>
                        <div className="space-y-2">
                            {beacons.map(beacon => (
                                <div key={beacon.id} className={`${beacon.color} text-lg font-medium p-2 rounded-md bg-gray-700/40 border border-teal-500/30`}>
                                    <i className="fas fa-lightbulb mr-2"></i>{beacon.name}
                                </div>
                            ))}
                        </div>
                        <div className="mt-4 flex gap-2">
                            <input
                                type="text"
                                placeholder="New Beacon Name"
                                value={newBeaconName}
                                onChange={(e) => setNewBeaconName(e.target.value)}
                                className="flex-grow p-2 rounded-md bg-gray-800 border border-gray-600 focus:outline-none focus:ring-1 focus:ring-blue-500"
                            />
                            <button onClick={addBeacon} className="bg-blue-600 hover:bg-blue-700 text-white p-2 rounded-md text-sm">Add Beacon</button>
                        </div>
                    </div>
                    <div>
                        <h3 className="text-2xl font-semibold text-blue-300 mb-3">Orbiting Intentions</h3>
                        <div className="space-y-2">
                            {intentions.map(intention => {
                                const beacon = beacons.find(b => b.id === intention.beaconId);
                                return (
                                    <div key={intention.id} className={`text-lg p-2 rounded-md bg-gray-700/40 border border-blue-500/30`}>
                                        <span className={`${beacon?.color || 'text-gray-400'} font-medium`}>
                                            <i className="fas fa-dot-circle mr-2"></i>{intention.name}
                                        </span>
                                        <span className={`ml-2 text-sm ${getPullStyle(intention.pull)}`}>({intention.pull} Pull)</span>
                                    </div>
                                );
                            })}
                        </div>
                         <div className="mt-4 flex flex-col sm:flex-row gap-2">
                            <input
                                type="text"
                                placeholder="New Intention Name"
                                value={newIntentionName}
                                onChange={(e) => setNewIntentionName(e.target.value)}
                                className="flex-grow p-2 rounded-md bg-gray-800 border border-gray-600 focus:outline-none focus:ring-1 focus:ring-blue-500"
                            />
                            <select
                                value={selectedBeaconForIntention}
                                onChange={(e) => setSelectedBeaconForIntention(e.target.value)}
                                className="p-2 rounded-md bg-gray-800 border border-gray-600 text-gray-100 focus:outline-none focus:ring-1 focus:ring-blue-500"
                            >
                                {beacons.map(beacon => (
                                    <option key={beacon.id} value={beacon.id}>{beacon.name}</option>
                                ))}
                            </select>
                            <button onClick={addIntention} className="bg-blue-600 hover:bg-blue-700 text-white p-2 rounded-md text-sm">Add Intention</button>
                        </div>
                    </div>
                </div>

                {/* Real-time Feedback (Static for prototype) */}
                <div className="bg-gray-800/50 p-4 rounded-lg border border-gray-700 mt-6">
                    <h3 className="text-xl font-semibold text-purple-300 mb-2">Real-time Alignment Feedback:</h3>
                    <p className="text-yellow-400"><i className="fas fa-triangle-exclamation mr-2"></i>Alignment Drift: Minor shifts detected in 'Personal Growth Project'.</p>
                    <p className="text-red-400"><i className="fas fa-compass-drafting mr-2"></i>Magnetic Distortion: Potential conflict between 'Web3 Grant Application' and 'Planetary Health' beacons.</p>
                </div>
            </div>

            {/* Radial Map Visualization (Simplified SVG for MVP) */}
            <div className="lg:w-1/2 flex items-center justify-center min-h-[300px] lg:min-h-0 bg-gray-800/50 rounded-lg border border-blue-700/30">
                <svg viewBox="0 0 200 200" className="w-full h-full max-w-sm max-h-sm">
                    {/* Center Beacon */}
                    <circle cx="100" cy="100" r="15" fill="#00BFFF" className="drop-shadow-lg animate-pulse" />
                    <text x="100" y="105" textAnchor="middle" fill="#FFFFFF" fontSize="8" fontWeight="bold">VISION</text>

                    {/* Core Beacons - Static positions for MVP */}
                    {beacons.slice(0, 3).map((beacon, index) => {
                        const angle = (index / 3) * 2 * Math.PI;
                        const x = 100 + 50 * Math.cos(angle);
                        const y = 100 + 50 * Math.sin(angle);
                        return (
                            <g key={beacon.id}>
                                <circle cx={x} cy={y} r="10" fill={beacon.color.replace('text-', '')} className="drop-shadow-md" />
                                <text x={x} y={y + 15} textAnchor="middle" fill={beacon.color.replace('text-', '')} fontSize="6">{beacon.name}</text>
                            </g>
                        );
                    })}

                    {/* Orbiting Intentions - Static positions for MVP */}
                    {intentions.slice(0, 4).map((intention, index) => {
                        const angle = (index / 4) * 2 * Math.PI + Math.PI / 8; // Offset slightly
                        const x = 100 + 80 * Math.cos(angle);
                        const y = 100 + 80 * Math.sin(angle);
                        return (
                            <g key={intention.id}>
                                <circle cx={x} cy={y} r="5" fill="#4B5563" stroke="#9CA3AF" strokeWidth="0.5" />
                                <text x={x} y={y + 10} textAnchor="middle" fill="#9CA3AF" fontSize="5">{intention.name}</text>
                            </g>
                        );
                    })}

                    {/* Example of a 'Magnetic Pull' line (static) */}
                    <line x1="100" y1="100" x2="160" y2="120" stroke="#FFD700" strokeWidth="1" opacity="0.6" strokeDasharray="2 2" />
                    <text x="140" y="125" fill="#FFD700" fontSize="5">Magnetic Pull</text>
                </svg>
            </div>
        </section>

        {/* Soul Dialogue Console */}
        <section id="soul-dialogue" className="glass-panel p-6 sm:p-10 rounded-xl shadow-lg border border-teal-700/30">
            <h2 className="text-3xl font-bold mb-6 neon-text text-center">🧠 Soul Dialogue Console</h2>
            <p className="text-gray-300 mb-6 text-center">
                An AI-powered journaling and reflective interface to help you re-center and gain clarity.
            </p>

            <div className="flex flex-col md:flex-row gap-6">
                <div className="flex-1">
                    <h3 className="text-2xl font-semibold text-teal-300 mb-3">Your Reflection</h3>
                    <form onSubmit={sendToSoulDialogue} className="flex flex-col gap-4">
                        <textarea
                            className="w-full h-40 p-4 rounded-md bg-gray-800 border border-gray-600 text-gray-100 focus:outline-none focus:ring-1 focus:ring-teal-500 resize-y"
                            placeholder="What are you being pulled toward? What feels misaligned this week?..."
                            value={journalEntry}
                            onChange={(e) => setJournalEntry(e.target.value)}
                            rows="6"
                            cols="50"
                        ></textarea>
                        <button
                            type="submit"
                            disabled={isLoadingAi || !journalEntry.trim()}
                            className="bg-teal-600 hover:bg-teal-700 text-white font-bold py-2 px-6 rounded-md transition-colors duration-200 disabled:opacity-50 disabled:cursor-not-allowed"
                        >
                            {isLoadingAi ? 'Reflecting...' : 'Send to AI Lighthouse'}
                        </button>
                    </form>
                </div>
                <div className="flex-1 bg-gray-800/50 p-6 rounded-lg border border-teal-700/30 overflow-y-auto max-h-[300px]">
                    <h3 className="text-2xl font-semibold text-blue-300 mb-3">Lighthouse Insights</h3>
                    {dialogueHistory.length === 0 && (
                        <p className="text-gray-400">Your AI lighthouse pulses will appear here.</p>
                    )}
                    <div className="space-y-4">
                        {dialogueHistory.map((entry, index) => (
                            <div key={index} className={`p-3 rounded-lg ${entry.type === 'user' ? 'bg-gray-700 self-end text-right' : 'bg-blue-900/50 self-start text-left'}`}>
                                <p className={`text-sm ${entry.type === 'user' ? 'text-gray-200' : 'text-blue-200'}`}>
                                    {entry.text}
                                </p>
                            </div>
                        ))}
                    </div>
                     {isLoadingAi && (
                        <p className="text-blue-400 mt-4 text-sm animate-pulse">Generating insights...</p>
                    )}
                </div>
            </div>
             <div className="mt-8 bg-gray-800/50 p-4 rounded-lg border border-gray-700">
                <h3 className="text-xl font-semibold text-purple-300 mb-2">Auto-summarized Themes (Future Feature)</h3>
                <p className="text-gray-400">Themes over time: Alignment with 'Sovereignty' (consistent), recurring pull towards 'Web3 Initiatives', occasional 'Energy Drain' related to 'Community Governance'.</p>
                <p className="text-gray-400 mt-2">Lighthouse Pulses generated: "Re-center on core values today", "Evaluate emotional resonance before committing", "Seek collaborative support for energy-intensive projects".</p>
            </div>
        </section>

        {/* Opportunity Field (Magnetic Scanner) */}
        <section id="opportunity-field" className="glass-panel p-6 sm:p-10 rounded-xl shadow-lg border border-blue-700/30">
            <h2 className="text-3xl font-bold mb-6 neon-text text-center">🌌 Opportunity Field: Magnetic Scanner</h2>
            <p className="text-gray-300 mb-6 text-center">
                An interactive stream of curated opportunities, semantically scored based on your Signal Map.
            </p>

            <div className="mb-6 flex flex-wrap gap-3 justify-center">
                <button onClick={() => setFilterTag('All')} className={`px-4 py-2 rounded-full text-sm font-medium ${filterTag === 'All' ? 'bg-blue-600 text-white' : 'bg-gray-700 hover:bg-gray-600'}`}>All</button>
                <button onClick={() => setFilterTag('Attract')} className={`px-4 py-2 rounded-full text-sm font-medium ${filterTag === 'Attract' ? 'bg-purple-600 text-white' : 'bg-gray-700 hover:bg-gray-600'}`}>Attract</button>
                <button onClick={() => setFilterTag('Ignore')} className={`px-4 py-2 rounded-full text-sm font-medium ${filterTag === 'Ignore' ? 'bg-red-600 text-white' : 'bg-gray-700 hover:bg-gray-600'}`}>Ignore</button>
                <button onClick={() => setFilterTag('Store')} className={`px-4 py-2 rounded-full text-sm font-medium ${filterTag === 'Store' ? 'bg-yellow-600 text-white' : 'bg-gray-700 hover:bg-gray-600'}`}>Store</button>
                <button onClick={() => setFilterTag('Re-align')} className={`px-4 py-2 rounded-full text-sm font-medium ${filterTag === 'Re-align' ? 'bg-green-600 text-white' : 'bg-gray-700 hover:bg-gray-600'}`}>Re-align</button>
            </div>

            <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
                {filteredOpportunities.map(op => (
                    <div key={op.id} className="bg-gray-800/50 p-5 rounded-lg border border-gray-700 hover:border-blue-500 transition-all duration-200">
                        <h3 className={`text-xl font-semibold mb-2 ${getRelevanceStyle(op.relevance)}`}>{op.title}</h3>
                        <p className="text-sm text-gray-400 mb-3">Source: {op.source} | Relevance: <span className="font-bold">{op.relevance.toFixed(2)}</span></p>
                        <div className="flex flex-wrap gap-2">
                            {op.tags.map((tag, idx) => (
                                <span key={idx} className={`text-xs font-medium px-3 py-1 rounded-full ${
                                    tag === 'Attract' ? 'bg-purple-800/50 text-purple-300' :
                                    tag === 'Ignore' ? 'bg-red-800/50 text-red-300' :
                                    tag === 'Store' ? 'bg-yellow-800/50 text-yellow-300' :
                                    tag === 'Re-align' ? 'bg-green-800/50 text-green-300' :
                                    'bg-gray-700/50 text-gray-300'
                                }`}>{tag}</span>
                            ))}
                        </div>
                    </div>
                ))}
            </div>

            {/* Animated currents visualization concept (CSS-based for MVP) */}
            <div className="relative h-24 mt-8 flex items-center justify-center overflow-hidden rounded-lg border border-blue-700/30 bg-gray-800/50">
                <div className="absolute inset-0 z-0 bg-gradient-to-r from-transparent via-blue-500/20 to-transparent animate-pulse-slow"></div>
                <p className="relative z-10 text-gray-400 text-center text-lg">
                    <i className="fas fa-satellite-dish mr-2 text-blue-400"></i> Scanning the opportunity field...
                </p>
            </div>
        </section>

        {/* Soulforge (Creative Ritual Space) */}
        <section id="soulforge" className="glass-panel p-6 sm:p-10 rounded-xl shadow-lg border border-teal-700/30">
            <h2 className="text-3xl font-bold mb-6 neon-text text-center">🔨 Soulforge: Creative Ritual Space</h2>
            <p className="text-gray-300 mb-6 text-center">
                A low-stimulation design sandbox for synthesizing project briefs and generating "lighthouse seeds".
            </p>

            <div className="flex flex-col gap-6">
                <div>
                    <h3 className="text-2xl font-semibold text-teal-300 mb-3">Synthesize Your Project Brief</h3>
                    <textarea
                        className="w-full h-32 p-4 rounded-md bg-gray-800 border border-gray-600 text-gray-100 focus:outline-none focus:ring-1 focus:ring-teal-500 resize-y"
                        placeholder="Describe your project, play with metaphors, or outline a system model..."
                        value={projectBriefInput}
                        onChange={(e) => setProjectBriefInput(e.target.value)}
                    ></textarea>
                    <button
                        onClick={createSoulSeed}
                        disabled={!projectBriefInput.trim()}
                        className="mt-4 bg-teal-600 hover:bg-teal-700 text-white font-bold py-2 px-6 rounded-md transition-colors duration-200 disabled:opacity-50 disabled:cursor-not-allowed"
                    >
                        Generate Lighthouse Seed <i className="fas fa-seedling ml-2"></i>
                    </button>
                </div>
                <div>
                    <h3 className="text-2xl font-semibold text-blue-300 mb-3">Your Lighthouse Seeds</h3>
                    {lighthouseSeeds.length === 0 ? (
                        <p className="text-gray-400">No seeds generated yet. Start crafting your project sparks!</p>
                    ) : (
                        <div className="space-y-3">
                            {lighthouseSeeds.map((seed, index) => (
                                <div key={index} className="bg-gray-800/50 p-4 rounded-lg border border-gray-700 flex justify-between items-center">
                                    <p className="text-gray-200 text-lg">{seed}</p>
                                    <div className="flex gap-2">
                                        <button title="Export to Markdown (Simulated)" className="text-gray-400 hover:text-white"><i className="fas fa-file-code"></i></button>
                                        <button title="Mint as NFT (Simulated)" className="text-gray-400 hover:text-white"><i className="fas fa-coins"></i></button>
                                    </div>
                                </div>
                            ))}
                        </div>
                    )}
                </div>
            </div>
             <div className="mt-8 bg-gray-800/50 p-4 rounded-lg border border-gray-700">
                <h3 className="text-xl font-semibold text-purple-300 mb-2">Design Card Deck (Concept)</h3>
                <p className="text-gray-400">Values, Constraints, Possibilities, Archetypes - for guiding your creative process.</p>
                <p className="text-gray-400 mt-2">Export options (Future): Markdown, Notion, GitHub Issue.</p>
            </div>
        </section>

        {/* Footer */}
        <footer className="py-10 text-center text-gray-400">
          <p>&copy; 2026 Magnetic Lighthouse with a Soul | A Coreweaver Labs Design.</p>
          <p className="mt-2">"Illuminating the Way for the Builders of Tomorrow."</p>
        </footer>
      </div>
    </div>
  );
}
