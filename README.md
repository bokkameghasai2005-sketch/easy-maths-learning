import { StatusBar } from 'expo-status-bar';
import React, { useMemo, useState } from 'react';
import {
  SafeAreaView,
  StyleSheet,
  Text,
  View,
  TouchableOpacity,
  Pressable,
  ScrollView,
} from 'react-native';

const difficulties = ['Easy', 'Medium', 'Hard'];

function randomInt(min, max) {
  return Math.floor(Math.random() * (max - min + 1)) + min;
}

function generateQuestion(level) {
  const operations = ['+', '-', '×', '÷'];
  const op = operations[level === 'Easy' ? 0 : level === 'Medium' ? randomInt(0, 2) : randomInt(0, 3)];

  let a = 0;
  let b = 0;
  let answer = 0;

  if (level === 'Easy') {
    a = randomInt(2, 12);
    b = randomInt(1, 9);
    answer = a + b;
    return {
      prompt: `${a} + ${b}`,
      answer,
      options: buildOptions(answer),
      operator: '+',
    };
  }

  if (level === 'Medium') {
    if (op === '+') {
      a = randomInt(10, 40);
      b = randomInt(5, 25);
      answer = a + b;
      return {
        prompt: `${a} + ${b}`,
        answer,
        options: buildOptions(answer),
        operator: '+',
      };
    }

    if (op === '-') {
      a = randomInt(20, 80);
      b = randomInt(5, 30);
      answer = a - b;
      return {
        prompt: `${a} - ${b}`,
        answer,
        options: buildOptions(answer),
        operator: '-',
      };
    }

    a = randomInt(3, 12);
    b = randomInt(2, 9);
    answer = a * b;
    return {
      prompt: `${a} × ${b}`,
      answer,
      options: buildOptions(answer),
      operator: '×',
    };
  }

  if (op === '+') {
    a = randomInt(35, 120);
    b = randomInt(20, 90);
    answer = a + b;
    return {
      prompt: `${a} + ${b}`,
      answer,
      options: buildOptions(answer),
      operator: '+',
    };
  }

  if (op === '-') {
    a = randomInt(60, 200);
    b = randomInt(10, 80);
    answer = a - b;
    return {
      prompt: `${a} - ${b}`,
      answer,
      options: buildOptions(answer),
      operator: '-',
    };
  }

  if (op === '×') {
    a = randomInt(6, 18);
    b = randomInt(5, 14);
    answer = a * b;
    return {
      prompt: `${a} × ${b}`,
      answer,
      options: buildOptions(answer),
      operator: '×',
    };
  }

  a = randomInt(20, 80);
  b = randomInt(2, 12);
  answer = Math.floor(a / b);
  const dividend = answer * b;
  return {
    prompt: `${dividend} ÷ ${b}`,
    answer,
    options: buildOptions(answer),
    operator: '÷',
  };
}

function buildOptions(correctAnswer) {
  const options = new Set([correctAnswer]);
  while (options.size < 4) {
    const offset = randomInt(-12, 20);
    const candidate = correctAnswer + offset;
    if (candidate >= 0) {
      options.add(candidate);
    }
  }

  return Array.from(options).sort(() => Math.random() - 0.5);
}

export default function App() {
  const [level, setLevel] = useState('Easy');
  const [question, setQuestion] = useState(() => generateQuestion('Easy'));
  const [score, setScore] = useState(0);
  const [streak, setStreak] = useState(0);
  const [bestScore, setBestScore] = useState(0);
  const [message, setMessage] = useState('Choose the correct answer!');
  const [selectedAnswer, setSelectedAnswer] = useState(null);

  const stats = useMemo(
    () => [
      { label: 'Score', value: score },
      { label: 'Streak', value: streak },
      { label: 'Best', value: bestScore },
    ],
    [score, streak, bestScore]
  );

  const startNewRound = (nextLevel = level) => {
    setSelectedAnswer(null);
    setMessage('Choose the correct answer!');
    setQuestion(generateQuestion(nextLevel));
  };

  const handleAnswer = (value) => {
    setSelectedAnswer(value);

    if (value === question.answer) {
      const newScore = score + 10;
      const newStreak = streak + 1;
      setScore(newScore);
      setStreak(newStreak);
      setBestScore((prev) => Math.max(prev, newScore));
      setMessage('Awesome! Correct answer!');
    } else {
      setScore((prev) => Math.max(0, prev - 5));
      setStreak(0);
      setMessage(`Nice try! Correct answer is ${question.answer}.`);
    }
  };

  const switchLevel = (newLevel) => {
    setLevel(newLevel);
    setQuestion(generateQuestion(newLevel));
    setSelectedAnswer(null);
    setMessage('Choose the correct answer!');
  };

  return (
    <SafeAreaView style={styles.container}>
      <StatusBar style="light" />
      <ScrollView contentContainerStyle={styles.content}>
        <View style={styles.headerCard}>
          <Text style={styles.title}>Easy Maths Learning</Text>
          <Text style={styles.subtitle}>Practice and improve your maths skills</Text>
        </View>

        <View style={styles.statsRow}>
          {stats.map((item) => (
            <View key={item.label} style={styles.statBox}>
              <Text style={styles.statLabel}>{item.label}</Text>
              <Text style={styles.statValue}>{item.value}</Text>
            </View>
          ))}
        </View>

        <View style={styles.levelRow}>
          {difficulties.map((item) => (
            <Pressable
              key={item}
              style={[styles.levelButton, level === item && styles.levelButtonActive]}
              onPress={() => switchLevel(item)}
            >
              <Text style={[styles.levelText, level === item && styles.levelTextActive]}>{item}</Text>
            </Pressable>
          ))}
        </View>

        <View style={styles.card}>
          <Text style={styles.cardLabel}>Question</Text>
          <Text style={styles.question}>{question.prompt}</Text>

          <View style={styles.optionGrid}>
            {question.options.map((option) => {
              const isCorrect = option === question.answer;
              const isSelected = selectedAnswer === option;

              let optionStyle = styles.optionButton;
              if (selectedAnswer !== null) {
                if (isCorrect) {
                  optionStyle = styles.optionButtonCorrect;
                } else if (isSelected) {
                  optionStyle = styles.optionButtonWrong;
                }
              }

              return (
                <TouchableOpacity
                  key={option}
                  style={[styles.optionButton, optionStyle]}
                  onPress={() => handleAnswer(option)}
                  disabled={selectedAnswer !== null}
                >
                  <Text style={styles.optionText}>{option}</Text>
                </TouchableOpacity>
              );
            })}
          </View>

          <Text style={styles.message}>{message}</Text>

          <TouchableOpacity style={styles.nextButton} onPress={() => startNewRound()}>
            <Text style={styles.nextButtonText}>Next Question</Text>
          </TouchableOpacity>
        </View>
      </ScrollView>
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#F3F4F6',
  },
  content: {
    padding: 20,
    paddingBottom: 40,
  },
  headerCard: {
    backgroundColor: '#7C3AED',
    borderRadius: 24,
    padding: 24,
    marginBottom: 20,
    shadowColor: '#000',
    shadowOpacity: 0.15,
    shadowRadius: 12,
    shadowOffset: { width: 0, height: 6 },
  },
  title: {
    color: '#fff',
    fontSize: 30,
    fontWeight: 'bold',
  },
  subtitle: {
    color: '#E9D5FF',
    fontSize: 16,
    marginTop: 6,
  },
  statsRow: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    marginBottom: 18,
  },
  statBox: {
    flex: 1,
    backgroundColor: '#fff',
    borderRadius: 16,
    paddingVertical: 16,
    marginHorizontal: 4,
    alignItems: 'center',
    shadowColor: '#000',
    shadowOpacity: 0.05,
    shadowRadius: 8,
    shadowOffset: { width: 0, height: 3 },
  },
  statLabel: {
    color: '#6B7280',
    fontSize: 12,
    marginBottom: 6,
  },
  statValue: {
    color: '#111827',
    fontSize: 26,
    fontWeight: 'bold',
  },
  levelRow: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    marginBottom: 20,
  },
  levelButton: {
    flex: 1,
    backgroundColor: '#E5E7EB',
    borderRadius: 12,
    paddingVertical: 12,
    marginHorizontal: 4,
    alignItems: 'center',
  },
  levelButtonActive: {
    backgroundColor: '#312E81',
  },
  levelText: {
    color: '#111827',
    fontWeight: '700',
  },
  levelTextActive: {
    color: '#fff',
  },
  card: {
    backgroundColor: '#fff',
    borderRadius: 24,
    padding: 20,
    shadowColor: '#000',
    shadowOpacity: 0.08,
    shadowRadius: 10,
    shadowOffset: { width: 0, height: 5 },
  },
  cardLabel: {
    color: '#8B5CF6',
    fontWeight: '700',
    fontSize: 14,
    textTransform: 'uppercase',
    letterSpacing: 0.8,
  },
  question: {
    marginTop: 12,
    marginBottom: 18,
    fontSize: 40,
    fontWeight: '800',
    color: '#111827',
    textAlign: 'center',
  },
  optionGrid: {
    flexDirection: 'row',
    flexWrap: 'wrap',
    justifyContent: 'space-between',
  },
  optionButton: {
    width: '48%',
    backgroundColor: '#F3F4F6',
    paddingVertical: 16,
    borderRadius: 16,
    marginBottom: 12,
    alignItems: 'center',
    borderWidth: 1,
    borderColor: '#E5E7EB',
  },
  optionButtonCorrect: {
    backgroundColor: '#A7F3D0',
    borderColor: '#10B981',
  },
  optionButtonWrong: {
    backgroundColor: '#FECACA',
    borderColor: '#EF4444',
  },
  optionText: {
    color: '#111827',
    fontSize: 24,
    fontWeight: '700',
  },
  message: {
    marginTop: 8,
    marginBottom: 20,
    textAlign: 'center',
    fontSize: 16,
    color: '#374151',
    fontWeight: '600',
  },
  nextButton: {
    backgroundColor: '#7C3AED',
    borderRadius: 14,
    paddingVertical: 14,
    alignItems: 'center',
  },
  nextButtonText: {
    color: '#fff',
    fontSize: 18,
    fontWeight: '700',
  },
});
