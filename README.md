/**
 * Differentially-private Aggregator (dp_aggregator.ts)
 *
 * Simple DP aggregator adding Laplace noise to numeric aggregates.
 * Usage: ts-node src/dp_aggregator.ts
 */

function laplaceNoise(scale: number) {
  // sample from Laplace(0, scale)
  const u = Math.random() - 0.5;
  return -scale * Math.sign(u) * Math.log(1 - 2 * Math.abs(u));
}

class DPAggregator {
  epsilon: number;
  sensitivity: number;

  constructor(epsilon = 1.0, sensitivity = 1.0) {
    this.epsilon = epsilon;
    this.sensitivity = sensitivity;
  }

  private laplaceScale() {
    return this.sensitivity / this.epsilon;
  }

  sum(values: number[]) {
    const trueSum = values.reduce((s, v) => s + v, 0);
    const noisy = trueSum + laplaceNoise(this.laplaceScale());
    return { trueSum, noisy };
  }

  avg(values: number[]) {
    if (values.length === 0) return { trueAvg: NaN, noisy: NaN };
    const { trueSum, noisy } = this.sum(values);
    return { trueAvg: trueSum / values.length, noisy: noisy / values.length };
  }
}

// demo
if (require.main === module) {
  const agg = new DPAggregator(0.5, 1.0);
  const data = Array.from({ length: 100 }, () => Math.floor(Math.random() * 10));
  console.log('True sum and noisy sum:', agg.sum(data));
  console.log('True avg and noisy avg:', agg.avg(data));
}
