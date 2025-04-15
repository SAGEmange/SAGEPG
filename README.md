function MainComponent() {
  const { data: user, loading: userLoading } = useUser();
  const [modelStats, setModelStats] = useState({
    currentVersion: "1.0.0",
    totalConversations: 0,
    positiveRatings: 0,
    negativeRatings: 0,
    averageRating: 0,
  });
  const [trainingProgress, setTrainingProgress] = useState({
    status: "idle",
    progress: 0,
    currentEpoch: 0,
    totalEpochs: 100,
  });
  const [conversations, setConversations] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  const [selectedConversations, setSelectedConversations] = useState([]);
  const [imagePrompt, setImagePrompt] = useState("");
  const [generatedImages, setGeneratedImages] = useState([]);
  const [imageLoading, setImageLoading] = useState(false);
  const [imageError, setImageError] = useState(null);

  useEffect(() => {
    const fetchData = async () => {
      setLoading(true);
      try {
        const response = await fetch("/api/mini-honoka/training/stats", {
          method: "POST",
        });
        if (!response.ok) {
          throw new Error("Failed to fetch training stats");
        }
        const data = await response.json();
        setModelStats(data.stats);
        setConversations(data.conversations || []);
      } catch (err) {
        console.error(err);
        setError("Failed to load training data");
      } finally {
        setLoading(false);
      }
    };

    if (user) {
      fetchData();
    }
  }, [user]);

  if (userLoading || loading) {
    return (
      <div className="min-h-screen bg-gray-900 flex items-center justify-center">
        <i className="fa fa-spinner fa-spin text-4xl text-white"></i>
      </div>
    );
  }

  if (!user) {
    return (
      <div className="min-h-screen bg-gray-900 flex items-center justify-center">
        <div className="text-center">
          <h1 className="text-3xl font-bold text-white mb-4">
            Access Restricted
          </h1>
          <a
            href="/account/signin"
            className="bg-blue-600 text-white px-6 py-2 rounded-lg hover:bg-blue-700"
          >
            Sign In
          </a>
        </div>
      </div>
    );
  }

  const generateImage = async () => {
    setImageLoading(true);
    setImageError(null);
    try {
      const response = await fetch(
        `/integrations/stable-diffusion-v-3/?prompt=${encodeURIComponent(imagePrompt)}`,
      );
      if (!response.ok) {
        throw new Error("Failed to generate image");
      }
      const data = await response.json();
      setGeneratedImages((prev) => [...prev, ...data.data]);
    } catch (err) {
      console.error(err);
      setImageError("Failed to generate image");
    } finally {
      setImageLoading(false);
    }
  };

  return (
    <div className="min-h-screen bg-gray-900 text-white p-6">
      <div className="max-w-7xl mx-auto">
        <div className="flex justify-between items-center mb-8">
          <h1 className="text-3xl font-bold">Mini Honoka Training Dashboard</h1>
          <div className="flex items-center space-x-4">
            <span className="bg-gray-800 px-4 py-2 rounded">
              Model Version: {modelStats.currentVersion}
            </span>
            <button className="bg-blue-600 px-4 py-2 rounded hover:bg-blue-700">
              <i className="fa fa-sync-alt mr-2"></i>
              Update Model
            </button>
          </div>
        </div>

        {error && (
          <div className="bg-red-500 bg-opacity-10 text-red-400 p-4 rounded-lg mb-8">
            {error}
          </div>
        )}

        <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6 mb-8">
          <div className="bg-gray-800 rounded-lg p-6">
            <h3 className="text-lg font-semibold mb-2">Total Conversations</h3>
            <div className="text-3xl font-bold">
              {modelStats.totalConversations}
            </div>
          </div>
          <div className="bg-gray-800 rounded-lg p-6">
            <h3 className="text-lg font-semibold mb-2">Positive Ratings</h3>
            <div className="text-3xl font-bold text-green-500">
              {modelStats.positiveRatings}
            </div>
          </div>
          <div className="bg-gray-800 rounded-lg p-6">
            <h3 className="text-lg font-semibold mb-2">Negative Ratings</h3>
            <div className="text-3xl font-bold text-red-500">
              {modelStats.negativeRatings}
            </div>
          </div>
          <div className="bg-gray-800 rounded-lg p-6">
            <h3 className="text-lg font-semibold mb-2">Average Rating</h3>
            <div className="text-3xl font-bold text-blue-500">
              {modelStats.averageRating.toFixed(2)}
            </div>
          </div>
        </div>

        <div className="bg-gray-800 rounded-lg p-6 mb-8">
          <h2 className="text-2xl font-bold mb-4">Training Progress</h2>
          <div className="mb-4">
            <div className="flex justify-between mb-2">
              <span>Progress: {trainingProgress.progress}%</span>
              <span>
                Epoch: {trainingProgress.currentEpoch}/
                {trainingProgress.totalEpochs}
              </span>
            </div>
            <div className="w-full bg-gray-700 rounded-full h-2.5">
              <div
                className="bg-blue-600 h-2.5 rounded-full"
                style={{ width: `${trainingProgress.progress}%` }}
              ></div>
            </div>
          </div>
          <div className="flex space-x-4">
            <button
              className={`px-4 py-2 rounded ${
                trainingProgress.status === "training"
                  ? "bg-red-600 hover:bg-red-700"
                  : "bg-green-600 hover:bg-green-700"
              }`}
            >
              {trainingProgress.status === "training" ? "Stop" : "Start"}{" "}
              Training
            </button>
          </div>
        </div>

        <div className="bg-gray-800 rounded-lg p-6">
          <div className="flex justify-between items-center mb-4">
            <h2 className="text-2xl font-bold">Training Image Generation</h2>
            <button
              onClick={generateImage}
              disabled={imageLoading || !imagePrompt}
              className="bg-blue-600 px-6 py-2 rounded hover:bg-blue-700 disabled:opacity-50 disabled:cursor-not-allowed"
            >
              {imageLoading ? (
                <i className="fa fa-spinner fa-spin mr-2"></i>
              ) : (
                <i className="fa fa-image mr-2"></i>
              )}
              Generate
            </button>
          </div>
          <div className="flex gap-4 mb-4">
            <input
              type="text"
              value={imagePrompt}
              onChange={(e) => setImagePrompt(e.target.value)}
              placeholder="Enter image description..."
              className="flex-1 bg-gray-700 rounded px-4 py-2 text-white"
            />
            {imageError && (
              <div className="bg-red-500 bg-opacity-10 text-red-400 p-4 rounded-lg mb-4">
                {imageError}
              </div>
            )}
          </div>

          {generatedImages.length > 0 && (
            <div className="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4">
              {generatedImages.map((url, index) => (
                <div key={index} className="relative group">
                  <img
                    src={url}
                    alt={`Generated ${index + 1}`}
                    className="w-full h-48 object-cover rounded-lg"
                  />
                  <div className="absolute inset-0 bg-black bg-opacity-50 opacity-0 group-hover:opacity-100 transition-opacity flex items-center justify-center rounded-lg">
                    <a
                      href={url}
                      target="_blank"
                      rel="noopener noreferrer"
                      className="text-white hover:text-blue-400"
                    >
                      <i className="fa fa-external-link-alt"></i>
                    </a>
                  </div>
                </div>
              ))}
            </div>
          )}
        </div>

        <div className="bg-gray-800 rounded-lg p-6">
          <div className="flex justify-between items-center mb-4">
            <h2 className="text-2xl font-bold">Training Conversations</h2>
            <button
              disabled={selectedConversations.length === 0}
              className="bg-blue-600 px-4 py-2 rounded hover:bg-blue-700 disabled:opacity-50 disabled:cursor-not-allowed"
            >
              Add Selected to Training ({selectedConversations.length})
            </button>
          </div>
          <div className="overflow-x-auto">
            <table className="w-full">
              <thead>
                <tr className="text-left border-b border-gray-700">
                  <th className="p-4">
                    <input
                      type="checkbox"
                      onChange={(e) => {
                        if (e.target.checked) {
                          setSelectedConversations(
                            conversations.map((c) => c.id),
                          );
                        } else {
                          setSelectedConversations([]);
                        }
                      }}
                    />
                  </th>
                  <th className="p-4">Date</th>
                  <th className="p-4">User</th>
                  <th className="p-4">Rating</th>
                  <th className="p-4">Length</th>
                  <th className="p-4">Actions</th>
                </tr>
              </thead>
              <tbody>
                {conversations.map((conversation) => (
                  <tr
                    key={conversation.id}
                    className="border-b border-gray-700 hover:bg-gray-700"
                  >
                    <td className="p-4">
                      <input
                        type="checkbox"
                        checked={selectedConversations.includes(
                          conversation.id,
                        )}
                        onChange={(e) => {
                          if (e.target.checked) {
                            setSelectedConversations([
                              ...selectedConversations,
                              conversation.id,
                            ]);
                          } else {
                            setSelectedConversations(
                              selectedConversations.filter(
                                (id) => id !== conversation.id,
                              ),
                            );
                          }
                        }}
                      />
                    </td>
                    <td className="p-4">
                      {new Date(conversation.date).toLocaleDateString()}
                    </td>
                    <td className="p-4">{conversation.user}</td>
                    <td className="p-4">
                      <span
                        className={`px-2 py-1 rounded ${
                          conversation.rating > 3
                            ? "bg-green-500 bg-opacity-20 text-green-400"
                            : "bg-red-500 bg-opacity-20 text-red-400"
                        }`}
                      >
                        {conversation.rating}/5
                      </span>
                    </td>
                    <td className="p-4">
                      {conversation.messageCount} messages
                    </td>
                    <td className="p-4">
                      <button className="text-blue-400 hover:text-blue-300">
                        <i className="fa fa-eye"></i>
                      </button>
                    </td>
                  </tr>
                ))}
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </div>
  );
}




